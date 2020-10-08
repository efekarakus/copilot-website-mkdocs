# env delete
```bash
$ copilot env delete [flags]
```

## What does it do?
`copilot env delete` deletes an environment from your application. If there are running applications in your environment, you first need to run [`copilot svc delete`](/docs/commands/svc-delete).

After you answer the questions, you should see the AWS CloudFormation stack for your environment gone.

## What are the flags?
```
-h, --help             help for delete
-n, --name string      Name of the environment.
    --profile string   Name of the profile.
    --yes              Skips confirmation prompt.
-a, --app string       Name of the application.
```

## Examples
Delete the "test" environment.
```bash
$ copilot env delete --name test --profile default
```
Delete the "test" environment without prompting.
```bash
$ copilot env delete --name test --profile default --yes
```