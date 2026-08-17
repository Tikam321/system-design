# LLM-Powered Jira Automation with Spring AI

## Overview

This service lets users describe what they want done in Jira using plain
natural language — e.g. *"push the due date for PROJ-101, PROJ-102, and
PROJ-103 out by 3 days"* — instead of manually opening each issue and
editing fields one at a time. The service interprets the request, maps it
to one or more Jira operations, executes them against the real Jira REST
API, and returns a plain-language summary of what happened.

It's built on **Spring Boot** and **Spring AI 2.0**, using Spring AI's
tool-calling (function-calling) support to connect the LLM directly to our
existing Jira integration code.

---

## Why we built it

Manually updating Jira issues in bulk is repetitive and error-prone:

- Sprint rollovers or scope changes often require updating due dates,
  statuses, or assignees across dozens of issues at once.
- Doing this by hand through the Jira UI is slow, and easy to get wrong
  (wrong issue, wrong date, missed ticket).
- Engineers and PMs were spending real time on what is fundamentally a
  translation problem: "I know what I want in plain English, I just need
  it turned into the right API calls."

The goal was to remove that translation step entirely — let people type
or say what they want, and have the system figure out *which* Jira
operations to call and *on which issues*, safely and in bulk.

**Impact:** this cut manual effort by roughly 95% for common bulk update
scenarios, and reduced operations that used to take many minutes of
manual clicking to a few seconds — even across 100+ issues at once.

---

## Why Spring AI

Before Spring AI, this kind of integration meant hand-rolling a lot of
plumbing: manually defining function/tool schemas as JSON, parsing the
model's function-call response, dispatching to the right Java method, and
feeding results back into the conversation — all boilerplate that has
nothing to do with the actual Jira logic.

Spring AI removes that boilerplate:

- **`@Tool` / `@ToolParam` annotations** turn a plain Java method into a
  callable LLM tool — no manual JSON schema writing.
- **`ChatClient`** handles sending the prompt, tool definitions, and
  managing the full round trip.
- **`ToolCallingAdvisor`** (auto-registered in Spring AI 2.0) sits in the
  advisor chain and does the actual tool-call loop: it intercepts the
  model's tool-call response, invokes the real Java method, and feeds the
  result back to the model — automatically, without us writing that loop
  by hand.
- It's provider-agnostic — the same tool definitions work whether the
  backing model is OpenAI, Anthropic, or a self-hosted/OpenAI-compatible
  Llama endpoint.

In short: Spring AI lets the *business logic* (talking to Jira) stay in
plain Spring service classes, while Spring AI owns the LLM orchestration
layer around it.

---

## Architecture — layers of the system

```
User query (natural language)
        │
        ▼
┌───────────────────────┐
│  Controller layer      │  REST endpoint, accepts the raw query
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│  Assistant service      │  Wraps ChatClient, sends prompt + tools
│  (JiraAssistantService) │
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│  Spring AI ChatClient   │  Talks to the LLM, manages the advisor chain
│  + ToolCallingAdvisor   │  (tool-call loop is automatic, not hand-written)
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│  Tool layer             │  @Tool-annotated methods: getIssue,
│  (JiraTools)             │  updateDueDate, bulkUpdateDueDate, etc.
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│  Jira client layer      │  Plain RestClient calls to Jira's REST API
│  (JiraClient)            │  (auth, endpoints, request/response mapping)
└───────────┬───────────┘
            ▼
        Jira REST API
```

Each layer has one job, which keeps the LLM-specific code isolated from
the actual Jira integration:

### 1. Jira client layer — `JiraClient`

Plain Spring `RestClient` wrapper around Jira's REST API. No LLM
awareness at all — this class would look identical in a non-AI
application. It handles auth (API token), request building, and response
parsing.

```java
@Component
public class JiraClient {

    private final RestClient restClient;

    public JiraClient(@Value("${jira.base-url}") String baseUrl,
                       @Value("${jira.email}") String email,
                       @Value("${jira.api-token}") String apiToken) {

        String auth = Base64.getEncoder()
                .encodeToString((email + ":" + apiToken).getBytes());

        this.restClient = RestClient.builder()
                .baseUrl(baseUrl)
                .defaultHeader(HttpHeaders.AUTHORIZATION, "Basic " + auth)
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();
    }

    public Map<String, Object> getIssue(String issueKey) {
        return restClient.get()
                .uri("/rest/api/3/issue/{issueKey}", issueKey)
                .retrieve()
                .body(Map.class);
    }

    public void updateDueDate(String issueKey, String dueDate) {
        Map<String, Object> body = Map.of("fields", Map.of("duedate", dueDate));
        restClient.put()
                .uri("/rest/api/3/issue/{issueKey}", issueKey)
                .body(body)
                .retrieve()
                .toBodilessEntity();
    }
}
```

### 2. Tool layer — `JiraTools`

This is where Spring AI enters the picture. Each method is a thin wrapper
around `JiraClient`, annotated with `@Tool` so the LLM knows it exists,
what it does, and what parameters it needs. The `description` fields are
what the model actually reads to decide *when* to call a given tool —
they matter as much as the code itself.

```java
@Component
public class JiraTools {

    private final JiraClient jiraClient;

    public JiraTools(JiraClient jiraClient) {
        this.jiraClient = jiraClient;
    }

    @Tool(description = "Fetch a Jira issue's details by its key, e.g. PROJ-123")
    public String getIssue(
            @ToolParam(description = "The Jira issue key, e.g. PROJ-123") String issueKey) {
        // fetches and formats issue details from JiraClient
    }

    @Tool(description = "Update the due date of a Jira issue")
    public String updateDueDate(
            @ToolParam(description = "The Jira issue key, e.g. PROJ-123") String issueKey,
            @ToolParam(description = "New due date in yyyy-MM-dd format") String dueDate) {
        // delegates to JiraClient.updateDueDate
    }

    @Tool(description = "Bulk update due dates for multiple Jira issues at once")
    public String bulkUpdateDueDate(
            @ToolParam(description = "List of Jira issue keys") List<String> issueKeys,
            @ToolParam(description = "New due date in yyyy-MM-dd format") String dueDate) {
        // loops over issueKeys, calls JiraClient per issue,
        // returns a consolidated success/failure report
    }
}
```

The `bulkUpdateDueDate` tool is what actually delivers the "100+ issues in
seconds" performance — it does the batching in plain Java, inside a
single tool call, rather than making the model call `updateDueDate`
once per issue (which would mean one LLM round trip per issue).

### 3. Assistant service layer — `JiraAssistantService`

Wires the tools into `ChatClient` and exposes a single `handle(query)`
method. This is the only class that knows an LLM is involved.

```java
@Service
public class JiraAssistantService {

    private final ChatClient chatClient;

    public JiraAssistantService(ChatClient.Builder chatClientBuilder, JiraTools jiraTools) {
        this.chatClient = chatClientBuilder
                .defaultTools(jiraTools)
                .build();
    }

    public String handle(String userQuery) {
        return chatClient.prompt()
                .user(userQuery)
                .call()
                .content();
    }
}
```

### 4. Controller layer — `JiraController`

Plain REST endpoint. No AI-specific logic — it just forwards the raw
query and returns the summary.

```java
@RestController
@RequestMapping("/api/jira")
public class JiraController {

    private final JiraAssistantService assistantService;

    public JiraController(JiraAssistantService assistantService) {
        this.assistantService = assistantService;
    }

    @PostMapping("/query")
    public String query(@RequestBody String naturalLanguageQuery) {
        return assistantService.handle(naturalLanguageQuery);
    }
}
```

---

## How a request actually flows

1. User submits: *"Push the due date for PROJ-101, PROJ-102, and
   PROJ-103 out by 3 days."*
2. `JiraAssistantService` sends the query to `ChatClient`, along with the
   registered tool schemas (`getIssue`, `updateDueDate`,
   `bulkUpdateDueDate`).
3. The model determines the intent maps to `bulkUpdateDueDate` and
   returns a tool-call request with the resolved issue keys and computed
   due date.
4. Spring AI's `ToolCallingAdvisor` intercepts that response, invokes
   `JiraTools.bulkUpdateDueDate(...)` in-process, and captures the
   result.
5. The result is fed back to the model, which produces a final
   natural-language summary — e.g. *"Updated due dates for PROJ-101,
   PROJ-102, and PROJ-103 to 2026-08-20."*
6. That summary is returned to the user through the controller.

All of steps 3–5 (the actual tool-call loop) are handled by Spring AI's
advisor chain — no manual JSON parsing or dispatch logic was written for
this.

---

## Where this improves productivity

| Before | After |
|---|---|
| Manually open each issue in the Jira UI, edit due date | Single natural-language request handles all issues |
| One-at-a-time updates, minutes per batch | Bulk tool call processes 100+ issues in seconds |
| Risk of missing or mis-editing an issue in a large batch | Deterministic Java loop in `bulkUpdateDueDate`, with per-issue success/failure reporting |
| Custom function-calling boilerplate to build and maintain | Spring AI's `@Tool` annotations + advisor chain handle orchestration |

---

## Possible next steps

- Add more tools: `bulkTransitionStatus`, `bulkAssign`, `addComment`,
  `linkIssues`.
- Add a confirmation step for destructive bulk operations (e.g. requiring
  explicit approval before applying changes to more than N issues).
- Swap `ChatModel` provider config only — no tool code changes needed —
  if we ever move off the current model provider, since tools are defined
  against Spring AI's abstraction, not a specific vendor SDK.
