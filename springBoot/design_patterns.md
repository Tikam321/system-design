1. mvc (model-view-controller) -> controller,service,repositoryand entity controller handle http request,service handlesbusiness logic and respository  handeles abtract data entities model the domain.;
2. template pattern -> adminuserserice(abstract), GovAdminService and KnoxBrityAmdinService override specific steps.
3. strategy patttern -> elasticservice -> elasticonservice/elasticoffservice (selected via conditinalOnProperty)
4. builder pattern -> lombok @builder on requestDTOclasses
5. chain of responsitbility -> requestFilter.java, TokenInterceptor,ExceptionResponseHandler) -> sequenctial processing of request from filter/interceptorand centralized exception handling.
6. statis factory emthod ->  API RESPONSE CLASS -> centralised creation of response objects
7. tempate method pattern -> AdminService(abstract) -> messengerAdminService and teamsAdminService concrte subclass which fills the abstract steps with their own steps .
8. singlton -> speng default bean scope provides single isntance per application context.
