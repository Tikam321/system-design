db schme architecture for policy management 
<img width="1334" height="424" alt="Screenshot 2026-04-04 at 10 32 54 AM" src="https://github.com/user-attachments/assets/2b02beb8-d6b1-418d-95f4-f6e9015204fc" />

- companylevel policy(Default level) apis 
- default feature policy
- admin/policy/checklist contain meetadata of policy
- admin/policy/stored -> contain values of plicies at different levels
- admin/policy/default/detail -> contain the defail of single policy

- company policy 
- admin/policy/checklist?companycode={}&suborgcode=ALL
- admin/policy/stored?companycode={}&suborgcode=ALL
- admin/policy/update?companycode={}&suborgcode=ALL

- suborgpolicy
- admin/policy/checklist?companycode={}&suborgcode={}
- admin/policy/stored?companycode={}&suborgcode={}
- admin/policy/update?companycode={}&suborgcode={}

- userpollicy
- view policy by user
- admin/policy/user?username={}&companycode={}&suborgcode={}
- admin/policy/user/detail?userid
- admin/policy/stored
- admin/policy/checklist
- admin/policy/user/update(/add)

- add user and assign them policy at user level(highest priority)
- admin/user/policy/add
- requestbody{
- userIds:[user1,user2,user3]
- storedPolicies:[
- {policyRuleId:{}, policyvalue:{}},
- {policyRuleId:{}, policyvalue:{}},
- {policyRuleId:{}, policyvalue:{}}
- ]
- }
- response: true(Added)

- view user bys policy
- admin/poliy/user/search?policyid={}?compnayCode&suborgcode
- response:[
- userIds:[user1,user2,user3...]
- }

userinfo
to check teh policies assigned to the users and their final value (final value accroding to precendence
highest to lowest default level > user level > org level > companylevel

admin/policy/checklist?userid&companycode?suborgcode
admin/policy/stored?userid&companycode?suborgcode



