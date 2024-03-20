# Attempt to address issue 6828

## Steps to reproduce

### Successfully adds preview URLs to Auth authorized domains

1. Add the Service Usage Consumer('roles/serviceusage.serviceUsageConsumer') role to the service account
1. Create a PR
   - This should correctly add the preview URL to Auth authorized domains

### Fails when adding preview URLs to Auth authorized domains

1. Remove the Service Usage Consumer('roles/serviceusage.serviceUsageConsumer') role from the service account
1. Create a PR
   - This does add the preview URL to Auth authorized domains

## Notes

Cannot get error logs from GitHub actions. Try to simulate locally using the ff conditions:

1. No user is logged in
   - Run `firebase logout`
1. Set the `GOOGLE_APPLICATION_CREDENTIALS` to point to your service account
   - Run `export GOOGLE_APPLICATION_CREDENTIALS=<path_to_service_account>`

Error logs from running `firebase hosting:channel:deploy --expires 1d <channel_id> --project some-testproject --debug`

```shell
[debug] [2024-03-20T10:52:18.140Z] >>> [apiv2][query] GET https://identitytoolkit.googleapis.com/admin/v2/projects/some-testproject/config [none]
[debug] [2024-03-20T10:52:18.853Z] <<< [apiv2][status] GET https://identitytoolkit.googleapis.com/admin/v2/projects/some-testproject/config 403
[debug] [2024-03-20T10:52:18.853Z] <<< [apiv2][body] GET https://identitytoolkit.googleapis.com/admin/v2/projects/some-testproject/config {"error":{"code":403,"message":"Caller does not have required permission to use project some-testproject. Grant the caller the roles/serviceusage.serviceUsageConsumer role, or a custom role with the serviceusage.services.use permission, by visiting https://console.developers.google.com/iam-admin/iam/project?project=some-testproject and then retry. Propagation of the new permission may take a few minutes.","status":"PERMISSION_DENIED","details":[{"@type":"type.googleapis.com/google.rpc.Help","links":[{"description":"Google developer console IAM admin","url":"https://console.developers.google.com/iam-admin/iam/project?project=some-testproject"}]},{"@type":"type.googleapis.com/google.rpc.ErrorInfo","reason":"USER_PROJECT_DENIED","domain":"googleapis.com","metadata":{"service":"identitytoolkit.googleapis.com","consumer":"projects/some-testproject"}}]}}
[warn] ⚠  hosting:channel: Unable to add channel domain to Firebase Auth. Visit the Firebase Console at https://console.firebase.google.com/project/some-testproject/authentication/providers


[debug] [2024-03-20T10:52:18.877Z] [hosting] unable to add auth domainHTTP Error: 403, Caller does not have required permission to use project some-testproject. Grant the caller the roles/serviceusage.serviceUsageConsumer role, or a custom role with the serviceusage.services.use permission, by visiting https://console.developers.google.com/iam-admin/iam/project?project=some-testproject and then retry. Propagation of the new permission may take a few minutes. {"name":"FirebaseError","children":[],"context":{"body":{"error":{"code":403,"message":"Caller does not have required permission to use project some-testproject. Grant the caller the roles/serviceusage.serviceUsageConsumer role, or a custom role with the serviceusage.services.use permission, by visiting https://console.developers.google.com/iam-admin/iam/project?project=some-testproject and then retry. Propagation of the new permission may take a few minutes.","status":"PERMISSION_DENIED","details":[{"@type":"type.googleapis.com/google.rpc.Help","links":[{"description":"Google developer console IAM admin","url":"https://console.developers.google.com/iam-admin/iam/project?project=some-testproject"}]},{"@type":"type.googleapis.com/google.rpc.ErrorInfo","reason":"USER_PROJECT_DENIED","domain":"googleapis.com","metadata":{"service":"identitytoolkit.googleapis.com","consumer":"projects/some-testproject"}}]}},"response":{"statusCode":403}},"exit":1,"message":"HTTP Error: 403, Caller does not have required permission to use project some-testproject. Grant the caller the roles/serviceusage.serviceUsageConsumer role, or a custom role with the serviceusage.services.use permission, by visiting https://console.developers.google.com/iam-admin/iam/project?project=some-testproject and then retry. Propagation of the new permission may take a few minutes.","status":403}

```

## Hypothesis

Adding the role 'roles/serviceusage.serviceUsageConsumer' to the service account during the `firebase init hosting:github` process should grant the service account permission to use enabled Google Cloud services(?).

-- Verified this behavior. Adding the role during `firebase init hosting:github` does seem to fix the issue
