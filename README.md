# Mike's AWS CLI Training Manual

#### First Published
Jun 6, 2025\
Using AWS CLI 2.27.45

# Table of Contents

* [Overview](#overview)
* [🛑 Do Not Use CLI Commands for Configuration](#-do-not-use-cli-commands-for-configuration)
* [The `credentials` and `config` files](#the-credentials-and-config-files)
* [Breakthroughs You’ll Need to Understand](#breakthroughs-youll-need-to-understand)
* [Specifying a Profile](#specifying-a-profile)
* [Using AWS SSO Profiles](#using-aws-sso-profiles)
* [Official AWS Documentation Links](#official-aws-ocumentation-inks)

# Overview

Understanding the AWS CLI is critical to using tools like terraform, kubectl, and anything else that uses AWS SDKs. The SDKs and the AWS CLI all share a config system.

The AWS CLI has a confusing config file structure, probably due to buildup over time while maintaining backward compatibility. Once you understand it, it’s not so arbitrary and confusing, so let’s break it down.

## Why You Should Care About All This

You’ll need an understanding of this stuff in order to create profile sections and understand how to specify a profile, since that’s how you:

* Identify yourself
* Target an AWS account

# 🛑 Do Not Use CLI Commands for Configuration

The AWS CLI has a bunch of commands you can use to edit your config files.

Technically that works, BUT I highly recommend against that.

That is like using a CLI or repl to edit code for a program you maintain. It is way easier to edit the files in a text editor.

# The `credentials` and `config` files

There are 2 files that the AWS CLI uses for configuration

* `credentials` is an older file, and was the only file originally
* `config` is newer, more explicit, and has support for more types of configuration

## Examples

`~/.aws/credentials file`

```
[default]
aws_access_key_id=<SomeIamUserAccessKey>
aws_secret_access_key=<SomeIamUserSecretKey>

[some-iam-user]
aws_access_key_id=<SomeIamUserAccessKey>
aws_secret_access_key=<SomeIamUserSecretKey>
```

`~/.aws/config` file

```
[default]
region=us-west-2
output=json

[sso-session yourcompany-access-portal]
sso_start_url = https://yourportalname.awsapps.com/start/#
# sso_region should be where Identity Center was configured
sso_region = us-west-2
sso_registration_scopes = sso:account:access

[profile dev]
sso_session = yourcompany-access-portal
sso_account_id = xxxxxxxxxxxx
sso_role_name = <SomeRoleName>

[profile central]
sso_session = yourcompany-access-portal
sso_account_id = xxxxxxxxxxxx
sso_role_name = <SomeRoleName>

[profile prod]
sso_session = yourcompany-access-portal
sso_account_id = xxxxxxxxxxxx
sso_role_name = <SomeRoleName>
```

# Breakthroughs You’ll Need to Understand

There are a few major mental breakthroughs you need to make to understand this system:

1. There are “section types”. We only care about a handful here.
   1. `global` aka `[default]`
   1. `profile`
   1. `sso-session`
1. You don’t actually need the `credentials` file at all
   1. Everything can be in `config` but not the other way around
   1. However, `credentials` takes precedence over `config` if there are duplicates
1. In both `credentials` and in `config`, there is one allowable global section
   1. It must be named `[default]`
   1. I have no idea why the name is not `[global]` after its section type name 🤷‍♂️
1. Major differences between the two files
   1. `credentials` is restricted to 2 section types: `global` and `profile`
   1. `credentials` requires you to leave off the section type names. Every section other than `[default]` is assumed to be of type `profile` and you may not explicitly declare the type name.
   1. `config`, conversely, requires you to explicitly specify section type names except for the `[default]` section

AWS documentation encourages any plain text keys to be put into the credentials file despite config being able to handle anything credentials can. Presumably, this is to encourage all secrets to be placed into their own file so it’s easier to share the config file without leaking credentials, but 🤷‍♂️. AWS does not have any kind of protections or guardrails around either file so they’re basically telling you to be precious with them manually.

# Specifying a Profile

There are 3 main ways to specify a profile:

## `AWS_PROFILE` Environment Variable

This is the best way.

The downside is that it requires you to remember to do it OR to add it to your shell profile.

### PowerShell

One Liner:

```$env:AWS_PROFILE="bluejay-dev"; aws sso login; aws sts get-caller-identity```

Persist:

```
$env:AWS_PROFILE="bluejay-dev"
aws sso login
aws sts get-caller-identity

etc...
```

### GitBash / Linux

One Liner:

```export AWS_PROFILE="bluejay-dev" && aws sso login && aws sts get-caller-identity```

Persist:

```
export AWS_PROFILE=bluejay-dev
aws sso login
aws sts get-caller-identity
aws s3 ls

etc...
```

## Rely on defaults

The main way to do this is to have a [default] section in either one of your files like this

```
[default]
aws_access_key_id=<SomeIamUserAccessKey>
aws_secret_access_key=<SomeIamUserSecretKey>
```

## Pass into every command

This works for the `aws` CLI itself, but won’t always work for tools that use SDKs. The tool has to explicitly support this style.

```
aws sts get-caller-identity --profile bluejay-dev
aws organizations describe-organization --profile bluejay-dev
aws s3 ls --profile studio-infra

etc...
```

# Using AWS SSO Profiles

With all the context above, logging in with SSO for tools or the `aws` CLI is basically just specifying a profile that is setup as an SSO user. Unlike IAM users, for SSO users, you’ll need to login first

```aws sso login```

The same profile specifying tricks above apply here (e.g. you can do aws sso login --profile my-profile to login to one explicitly).

If you you haven’t logged in or your sso session token has expired, you’ll get this error message:

```Error loading SSO Token: Token for wildlight-access-portal does not exist```

# Official AWS Documentation Links

* [Installation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
* [Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
