# arruko.aws_sts

[![Build Status](https://travis-ci.org/arruko/aws-sts.svg?branch=master)](https://travis-ci.org/arruko/aws-sts)

# AWS STS Role

This Ansible [role](https://galaxy.ansible.com/arruko/aws_sts) allows a user to assume a given role, generating temporary security credentials that can be used to assume the role. I've based on original role to add `awscli` configuration before role starts to authenticate with AWS.

## Requirements

- Python 2.7
- PIP package manager (**easy_install pip**)
- Ansible 2.4 or greater (**pip install ansible**)
- AWS CLI (**pip install awscli**)

## Setup

The recommended approach to use this role is an Ansible Galaxy requirement to your Ansible playbook project.

## Role Variables

Please refer to the [defaults file](/defaults/main.yml) for an up to date list of input parameters.

```
# MFA arn for awscli authentication
aws_mfa_arn: ""

# AWS account id 
aws_account_id: ""

# AWS admin role to assume
aws_admin_role: ""

# Name for ~/.aws/credentials profile
aws_credential_profile_name: ""

# AWS Key id for awscli
awscli_credentials_key_id: ""

# AWS access key for awscli
awscli_credentials_access_key: ""
```

### Installing using Ansible Galaxy

To set this role up as an Ansible Galaxy requirement, first create a `requirements.yml` file in a subfolder called `roles` and add an entry for this role.  See the [Ansible Galaxy documentation](http://docs.ansible.com/ansible/galaxy.html#installing-multiple-roles-from-a-file) for more details.

```
# Example requirements.yml file
- name: arruko.aws_sts
  version: "1.0.4"
```

Once you have created `roles/requirements.yml`, you can install the role using the `ansible-galaxy` command line tool.

```
$ ansible-galaxy install --role-file=roles/requirements.yml --roles-path=./roles/ --force
$ git commit -a -m "Added aws-sts 1.0.4 role"
```

To update the role version, simply update the `requirements.yml` file and re-install the role as demonstrated above.

## Usage

NOTE: This role will overwrite your `~/.aws` folder and files during role execution to config `awscli` according to [defaults file](/defaults/main.yml) values.

### Inputs

The STS role relies on the following inputs:

- `Sts.Role` (Mandatory) - this variable must be provided by the calling playbook, representing the ARN of the role to assume

- `Sts.SessionName` (Optional) - a name for the temporary session token that is generated

- `Sts.Disabled` (Optional) - disables all actions of this role.  Useful for long running playbooks that would be affected by the duration (maximum 60 minutes) of using STS token.

- `Sts.Region` (Optional) - the target AWS regions.  Alternatively you can set the AWS region using the `AWS_DEFAULT_REGION` environment variable.  If this is not in your environment, the playbook defaults to `ap-southeast-2`.

- AWS credentials - you must configure the environment such that your credentials are available to assume the role.  For example, you can set an access key and secret key via environment variables, or configure a profile via environment variables, or rely on an EC2 instance profile if running in AWS.  A dictionary called `Sts.Credentials` is output by this module, which sets up the appropriate configuration with AWS STS token settings.

### Outputs

If the assume role operation is successful, the temporary credentials issued by AWS are registered to a variable called `Sts.Credentials`:

- `Sts.Credentials.AWS_DEFAULT_REGION`
- `Sts.Credentials.ACCESS_KEY`
- `Sts.Credentials.ACCESS_KEY_ID`
- `Sts.Credentials.AWS_SECRET_KEY`
- `Sts.Credentials.AWS_SECRET_ACCESS_KEY`
- `Sts.Credentials.AWS_SECURITY_TOKEN`

You should call this role from a dedicated play, and then define your subsequent playbook tasks in separate plays.  This allows the `Sts.Credentials`variable to be used to configure the environment of your remaining tasks.


## Examples

The following demonstrates how to call this role from a playbook:

> You cannot use the dot notation syntax `Sts.Role` with the `vars` syntax as demonstrated in the example below.

```yaml
- name: Assume Role Play
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    Sts:
      Role: "arn:aws:iam::123456789:role/admin"
      SessionName: testAssumeRole
      Region: us-west-2
  roles:
    - aws-sts
```

The following shows the recommended play configuration to use the temporary credentials issued:

```yaml
...
...
- name: My Playbook
  hosts: localhost
  connection: local
  environment: "{{ Sts.Credentials }}"
  tasks:
   - ...
   - ...
...
...
```
## Errata

No known issues.

## TODO

- Unit Testing with Molecule

## License

This project is licensed under the terms of the [MIT License](/LICENSE)
