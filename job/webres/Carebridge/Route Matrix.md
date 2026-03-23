

| route                                               | Protocol | controller action                                    | permission mechanism | ownership enforcement | tenant scoping | risk level | remediation recommendation |
| --------------------------------------------------- | -------- | ---------------------------------------------------- | -------------------- | --------------------- | -------------- | ---------- | -------------------------- |
| login                                               | get      | AuthenticatedSessionController@create                |                      |                       |                |            |                            |
| login                                               | post     | AuthenticatedSessionController@store                 |                      |                       |                |            |                            |
| forgot-password                                     | get      | PasswordResetLinkController@create                   |                      |                       |                |            |                            |
| forgot-password                                     | post     | PasswordResetLinkController@forgotPassword           |                      |                       |                |            |                            |
| reset-password/{token}                              | get      | NewPasswordController@create                         |                      |                       |                |            |                            |
| reset-password-request/{email}/{displayFirstLogin?} | get      | NewPasswordController@requestSent                    |                      |                       |                |            |                            |
| reset-password                                      | post     | NewPasswordController@resetPassword                  |                      |                       |                |            |                            |
| reset-password-store                                | post     | NewPasswordController@store                          |                      |                       |                |            |                            |
| confirm-password                                    | get      | ConfirmablePasswordController@show                   |                      |                       |                |            |                            |
| confirm-password                                    | post     | ConfirmablePasswordController@store                  |                      |                       |                |            |                            |
| password                                            | put      | PasswordController@update                            |                      |                       |                |            |                            |
| logout                                              | post     | AuthenticatedSessionController@destroy               |                      |                       |                |            |                            |
| /health-check                                       | get      | Closure                                              |                      |                       |                |            |                            |
| /forgot-password                                    | post     | PasswordResetLinkController@store                    |                      |                       |                |            |                            |
| /forgot-password                                    | get      | Closure                                              |                      |                       |                |            |                            |
| /invitation/{token}                                 | get      | Closure                                              |                      |                       |                |            |                            |
| /2fa                                                | get      | TwoFAController@index                                |                      |                       |                |            |                            |
| /2fa                                                | post     | TwoFAController@store                                |                      |                       |                |            |                            |
| 2fa/reset                                           | get      | TwoFAController@resend                               |                      |                       |                |            |                            |
| logout                                              | post     | AuthenticatedSessionController@destroy               |                      |                       |                |            |                            |
| /applications-enquiry/{status}/{uuid}/{decision}    | get      | ApplicationController@providerStatusDecision         |                      |                       |                |            |                            |
| /enquiry-acknowledge/{status}/{uuid}/{decision}     | get      | ApplicationController@providerEnquiryAcknowledgement |                      |                       |                |            |                            |
| /profile                                            | get      | Closure                                              |                      |                       |                |            |                            |
| /                                                   | get      | IndexController@index                                |                      |                       |                |            |                            |
| /home                                               | get      | HomeController@index                                 |                      |                       |                |            |                            |
| /failed-jobs                                        | get      | FailedJobController@index                            |                      |                       |                |            |                            |
| /notifications/{notification}/mark-as-read          | post     | NotificationController@markAsRead                    |                      |                       |                |            |                            |
| /users/{id?}                                        | get      | UserController@index                                 |                      |                       |                |            |                            |
| /users                                              | post     | UserController@store                                 |                      |                       |                |            |                            |
| /users/{id}                                         | put      | UserController@update                                |                      |                       |                |            |                            |
| /user-profile                                       | put      | UserController@userProfile                           |                      |                       |                |            |                            |
| /users/{id}                                         | delete   | UserController@destroy                               |                      |                       |                |            |                            |
| /map                                                | get      | MapController@index                                  |                      |                       |                |            |                            |
| /companies/{id?}                                    | get      | CompanyController@index                              |                      |                       |                |            |                            |
| /companies                                          | post     | CompanyController@store                              |                      |                       |                |            |                            |
| /companies/{id}                                     | post     | CompanyController@update                             |                      |                       |                |            |                            |
| /companies/{id}                                     | delete   | CompanyController@destroy                            |                      |                       |                |            |                            |
| /api/companies/by-type/{type}                       | get      | CompanyController@getByType                          |                      |                       |                |            |                            |
| /facilities/{id?}/{vacancy?}                        | get      | FacilityController@index                             |                      |                       |                |            |                            |
| /facilities                                         | post     | FacilityController@store                             |                      |                       |                |            |                            |
| /facilities/{id}                                    | put      | FacilityController@update                            |                      |                       |                |            |                            |
| /facilities/{id}                                    | delete   | FacilityController@destroy                           |                      |                       |                |            |                            |
| /vacancies                                          | get      | VacancyController@index                              |                      |                       |                |            |                            |
| /facilities/{facility}/vacancy                      | put      | VacancyController@update                             |                      |                       |                |            |                            |
| /facilities/{facility}/vacancy/{vacancy?}           | delete   | VacancyController@destroy                            |                      |                       |                |            |                            |
| /facilities/{facility}/vacancy-refresh              | put      | VacancyController@refresh                            |                      |                       |                |            |                            |
| /application-files/{applicationFile}                | get      | ApplicationFileController@download                   |                      |                       |                |            |                            |
| /application-files/{applicationFile}                | delete   | ApplicationFileController@delete                     |                      |                       |                |            |                            |
| /applications                                       | get      | ApplicationController@index                          |                      |                       |                |            |                            |
| /test                                               | get      | ApplicationController@test                           |                      |                       |                |            |                            |
| /applications/{id}                                  | get      | ApplicationController@index                          |                      |                       |                |            |                            |
| /applications/new                                   | get      | ApplicationController@index                          |                      |                       |                |            |                            |
| /applications/new                                   | post     | ApplicationController@store                          |                      |                       |                |            |                            |
| /applications-enquiry                               | post     | ApplicationController@mapSubmitEnquiry               |                      |                       |                |            |                            |
| /applications/edit/{id}                             | post     | ApplicationController@update                         |                      |                       |                |            |                            |
| /enquiries                                          | get      | EnquiryController@index                              |                      |                       |                |            |                            |
| /enquiries/{id}                                     | get      | EnquiryController@index                              |                      |                       |                |            |                            |
| /application/{id}/transition/{statusId}             | post     | ApplicationController@transition                     |                      |                       |                |            |                            |
| /applications/{id}/patient-transfer                 | post     | ApplicationController@patientTransfer                |                      |                       |                |            |                            |
| /enquiries/{id}/transition/{statusId}               | post     | EnquiryController@transition                         |                      |                       |                |            |                            |
| /enquiry-history/{enquiryHistoryId}/undo            | post     | EnquiryController@undoTransition                     |                      |                       |                |            |                            |
| /requested-enquiries/update-enquiry-status          | post     | ApplicationController@enquiryStatusUpdateByProvider  |                      |                       |                |            |                            |
| /long-application                                   | get      | Closure                                              |                      |                       |                |            |                            |
| /api/type-of-care/{accommodationType}/solo-facility | get      | TypeOfCareController@showSoloFacility                |                      |                       |                |            |                            |



I need to complete the following task:

Goal:Do the

- Identify which controller actions have:
    - role/permission checks
    - ownership checks
    - organisation scoping
    - Insecure Direct Object Access Risk

Deliverable:

- A route/action matrix with:
    - route
    - controller action
    - permission mechanism
    - ownership enforcement
    - tenant scoping
    - risk level
    - remediation recommendation
      
      
To that end, I have created a table of the available routes in the .agents/plans/permissions-plan.md file. Please fill in the first two rows of the table (the login routes). 