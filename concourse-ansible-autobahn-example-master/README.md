# concourse-ansible-autobahn-example

**Problem**: Your `ci.comcast.net` Concourse CI/CD Ansible jobs need SSH connectivity to resources hosted in a public AWS VPC and only reachable via traversal of the public internet.

**Solution**: [autobahn.comcast.com](https://autobahn.comcast.com)!

## How To

The following instructions are based on the [Project Autobahn Guide](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/index.md) and, more specifically, the [Project Autobahn Administrator Guide](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Administrator_Guide.md).

If you run into issues with what's documented here, please refer to the more-official documentation on which these instructions are based.

### 1. Enroll in Autobahn

1. log into [autobahn.comcast.com](https://autobahn.comcast.com).
2. Visit the [Public SSH key](https://autobahn.comcast.com/key)
3. Paste your public key in the text box (typically, this is stored in on your laptop in a `~/.ssh/id_rsa.pub` file). After pasting your key, click "Save" to save your key to Autobahn.

### 2. Request that a new Autobahn team be created

The [Autobahnn homepage](https://autobahn.comcast.com/) lists your teams. If your team does not appear in the list, you may need to onboard the team to Autobahn.

Create a _team_. Email `T&P-ProductSecurityandPrivacyDevelopment@comcast.com` requesting your team be created. For example:

> Please create a new team named "MY_TEAM_NAME" in autobahn. My NTID is 'MY_NT_ID' and I have enrolled in project autobahn.

Once a team has been created, the Autobahn team will set an enrolled user as **team owner**. Team owners can add and remove other users to/from the team, create sub-applications, and make service accounts.

**Note**: The `ci/pipeline.yml` example contained in this repository uses the `Cubits` team.

### 3. Onboard to [Comcast Secrets as a Service](https://security.sys.comcast.net/How_Do_I/Secrets_as_a_Service/index.md)

1. Create a `vault-admins` [team](https://help.github.com/en/articles/organizing-members-into-teams) in your `github.comcast.com` [org](https://help.github.com/en/articles/about-organizations) (Really, it's up to you to choose a good team name, though `vault-admins` or `secrets-as-a-service-admins` is self-explanatory)
2. Add the appropriate org members to the `vault-admins` team via the `github.comcast.com` web UI.
3. Email `T&P-ProductSecurityandPrivacyDevelopment@comcast.com` to request a namespace be created:
    > Hello, I would like a TPX Vault namespace created for `MY_GHE_ORG`. Initially, the `MY_GHE_ORG` `vault-admins` team should have read/write access to the `MY_GHE_ORG/*` secrets.
4. Once your initial Vault policies are created, you can view them (and create [pull requests](https://help.github.com/en/articles/about-pull-requests) to modify them) at [github.comcast.com/TP-PSP/VaultPolicies](https://github.comcast.com/TP-PSP/VaultPolicies)

### 4. Create a [Autobahn service account](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Service_Account.md) via the `autobahn.comcast.com` web UI (you need to be the team admin to do this).

1. Give your service account a **Name** such as `svc<YOUR_GHE_ORG>atb`
2. Give it an **App Role Name**  such as `ci` for use in an associated TPX Vault policy
3. Specify `YOUR_GHE_ORG` as the **GitHub Org** with which to associate the service account
4. Paste the **Public Key** part of an SSH key pair used by the service account (If you don't already have an SSH keypair to associate with this service account, you may need to [create one](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Enrollment.md#ssh-keygen))
5. Add a **Certificate TTL** such as `21600` specifying the number of seconds an Autobahn certificate can live (an Autobahn certificate is used to authenticate to Autobahn; more on this later)
6. Click "Save"

**Note**: The `ci/pipeline.yml` example contained in this repository uses the `Cubits` team's [svcubitsab](https://autobahn.comcast.com/service-accounts) service account

### 5. Create an Autobahn **application** and **principal**

1. Visit the [autobahn.comcast.com](https://autobahn.comcast.com) homepage
2. Click the "Manage" link associated with your team
3. Click "Create New Application"
4. Name your application something like `ci` and click "Create"
5. Click the "Manage" link associated with your `ci` application
6. Click "Add new principal"
7. Name your **principal** `admin`

**Note**: The `ci/pipeline.yml` example contained in this repository uses the `Cubits` team's [cubits-ci-apps](https://autobahn.comcast.com/teams/197/applications/667/edit) application and a [cubits.cubits-ci-apps.admin](https://autobahn.comcast.com/teams/197/applications/667/principals/1694/edit) principal.

### 6. Map your **service account** to your `admin` **principal**

1. Click the "Manage" link associated with your **principal** (From the `autobahn.comcast.com` homepage, click "Manage" your team, click "Manage" your **application**, and click the "Manage Principal" associated with your `admin` principal)
2. Search for your `svc<YOUR_GHE_ORG>atb` service account (or whatever name you chose to name your service account)
3. Map the service account to your `admin` principal
4. Visit [autobahn.comcast.com/service-accounts](https://autobahn.comcast.com/service-accounts) and click the "Pull Request" button associated with your service account. This creates a pull request against the [TP-PSP/VaultPolicies](https://github.comcast.com/TP-PSP/VaultPolicies) repository. When the pull request is reviewed and merged, you're ready to start using your service account to request Autobahn certificates and use Autobahn to SSH to your infrastructure.

**Note**: The `ci/pipeline.yml` example contained in this repository uses the `Cubits` team's [VaultPolicy](https://github.comcast.com/TP-PSP/VaultPolicies/tree/master/organizations/cubits).

### 7. Configure your servers for use with Autobahn

1. [Configure](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Jumpservers.md) your security groups and/or ACLs to allow connectivity from `jump.autobahn.comcast.com` servers
2. [Configure your servers](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Server_Install.md) for use with Autobahn. Note that there is a [tp-ansible/ansible-autobahn](https://github.comcast.com/tp-ansible/ansible-autobahn) Ansible role to automate this.

**Note**: The `ci/pipeline.yml` example contained in this repository uses the `Cubits` team's public VPC AWS host, `54.164.36.2`.

### 8. Request an **Autobahn certificate**

Each time your service account authenticates to the `jump.autobahn.comcast.com` servers, you'll need to generate an Autobahn certificate for authenticating to Autobahn.

This can by done by a manual sequence of [API requests](https://security.sys.comcast.net/How_Do_I/Project_Autobahn/Service_Account.md#requesting-a-cert-for-your-autobahn-service-account). However, tools like [julien](https://github.comcast.com/sst/julien) and a [Concourse get_autobahn_certificate.yml task](https://github.comcast.com/concourse/recipes/blob/master/tasks/misc/get_autobahn_certificate.yml) offer convenience utilities that automate the API transactions.

**Note**: See the [run-ansible-playbook-through-autobahn-using-recipes]() configured via the `ci/pipeline.yml` `run-ansible-playbook-through-autobahn-using-recipes` job for an example of executing an Ansible playbook against an AWS host through Autobahn.
