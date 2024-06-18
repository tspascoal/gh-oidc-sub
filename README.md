# GH oidc-sub

> Manage GitHub Actions OIDC subject custom templates within an organization or repository.

## Why?

OpenID connect (ODIC) is a standard for authentication and authorization. OIDC support in GitHub Actions enables secure cloud deployments using short-lived tokens that are automatically rotated for each deployment.

You can configure ODIC to configure the [subject claim format](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect#customizing-the-subject-claims-for-an-organization-or-repository) within the OIDC tokens, by defining a customization template at either org or repo levels. Once the configuration is completed, the new OIDC tokens generated during each deployment will follow the custom format.

This enables organization & repository admins to standardize OIDC configuration across their cloud deployment workflows that suits their compliance & security needs.

## Installation

### Prerequisites

- `Bash`
- `jq` (optional but recommended)
- `curl`

### gh cli extension

You can install `oidc-sub` as a [gh cli](https://github.com/cli/cli) extension!

```console
gh extensions install tspascoal/gh-oidc-sub
```

#### Verify installation

```console	
gh oidc-sub --help
```

## Usage

### Operations

This extension supports the following operations:

- get
- set
- use-default (or usedefault)
- list-claims
- list-repos

To understand the semantics of claim customizaiton, please read [Customizing the subject claims for an organization or repository](https://docs.github.com/en/enterprise-cloud@latest/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect#customizing-the-subject-claims-for-an-organization-or-repository).

Pay special attention of the opt in semantics for repository customizations. Setting the organization may not be enough, quoting from GitHub docs:

> [!Note]
> When the organization template is applied, it will not affect any action workflows in existing repositories that already use OIDC. For existing repositories, as well as any new repositories that are created after the template has been applied, the repository owner will need to opt-in to receive this configuration, or alternatively could apply a different configuration specific to the repo. For more information, see "[Set the customization template for an OIDC subject claim for a repository.](https://docs.github.com/en/enterprise-cloud@latest/rest/actions/oidc#set-the-customization-template-for-an-oidc-subject-claim-for-a-repository)"

#### list-claims

Gets the list of sub claims that can used in the template.

You can read more about the claims and their semantic in the [Understanding the OIDC token](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect#understanding-the-oidc-token).

#### get

Get the current OIDC subject configuration for an organization or repository.

For organizations it will throw an error if no claims have ever been customized.

For repositories it will return if the default subject is used and the list of claims.

The output will always be the full result of the API call (in JSON), you can use the `--jq` parameter with a `jq` expression to filter the output (no need for `jq` to be installed).

#### Examples

<details>

<summary>
Get the organization custom OIDC claims for an organization that never had subject claims customized.
</summary>

```console
$ gh oidc-sub get --org mona
{
  "message": "No Actions OIDC custom sub claim template found for the organization tspascoal-demo2",
  "documentation_url": "https://docs.github.com/rest/actions/oidc#get-the-customization-template-for-an-oidc-subject-claim-for-an-organization"
}
gh: No Actions OIDC custom sub claim template found for the organization tspascoal-demo2 (HTTP 404)
```
</details>
<details>
<summary>
Get the organization custom OIDC claims for an organization with subject claims customized.
</summary>

```console
$ gh oidc-sub get --org mona
{
  "include_claim_keys": [
    "repo"
  ]
}
```
</details>

<details>
<summary>
Get repository custom OIDC claims for an repo with subject claims customized, returning only the claims.
</summary>
  
```console
$ gh oidc-sub get --repo mona/repo
{
  "use_default": false,
  "include_claim_keys": [
    "repo"
  ]
}
```

</details>
<details>
<summary>
Get repository custom OIDC claims for an repo with subject claims customized, returning only the claims.
</summary>
 
```console
$ gh oidc-sub get --repo mona/repo --jq '.include_claim_keys'
[ "repo" ]
```
</details>

#### set

Set the OIDC subject configuration for an organization or repository.

For repository the `--subs` paramemeter is optional. If you don't set the list of claims, we will opt out out of the default claims and organization claims will be used instead, if you do specificy the subs than those will be used instead of the organization claims.

> The claims keys are not validated so be sure you are not mispelling any claim.

##### Examples

<details>
<summary>
Set custom claims template for an organization
</summary>

```console
$ gh oidc-sub set --org mona --subs "org,repo,job_workflow_ref"
{}
```
</details>

<details>
<summary>
Set custom claims template for a repository
</summary>

```console
$ gh oidc-sub set --repo mona/repo --subs "org,repo"
{}
```
</details>
<details>
<summary>
Configure repository to use organization custom claims template
</summary>

```console
$ gh oidc-sub set --repo mona/repo
{}
```
</details>

#### use-default

If you want to revert a customization, you can use this command.

For organizations it sets the customization subject claims to `repo,context`

For repositories it disables the customization subject claims and configures it to use the default ones (`use_default: true`)

##### Examples

<details>
<summary>
Configure organization customization subject claims to the default value
</summary>

```console
$ gh oidc-sub use-default --org mona
{}

$ gh oidc-sub get --org mona
{
  "include_claim_keys": [
    "repo",
    "context"
  ]
}
```
</details>
<details>
<summary>
Configure repository customization subject claims to use the defaults
</summary>

```console
$ gh oidc-sub use-default --repo mona/repo
{}

$ gh oidc-sub get --repo mona/repo
{
  "use_default": true
}
```
</details>

#### list-repos

List the current configuration for all repositories in a given organization. This will give a full view which repos are using the default configuration and which ones are using customized claims.

The output will be a TSV with:
- Repository
- If it uses default claims
- Claims list (comma separated)

#### Examples

<details>

<summary>Get organization repositories oidc claims template customization</summary>

```console
$ gh oidc-sub list-repos --org mona
repository	use default	claims
tspascoal-oidc/customized	false	repo,ref
tspascoal-oidc/usingdefault	true
```

</details>

### Parameters

Compatible with [GitHub Enterprise Server](https://github.com/enterprise). ([see](https://cli.github.com/manual/) for more details)

```text

Usage: 
    <command> [options]:
        get (--org ORG | --repo owner/repo) [--jq JQ_FILTER]
        set (--org ORG | --repo owner/repo) --subs "SUBS LIST"
        use-default (--org ORG | --repo owner/repo) (Note: usedefault is also supported)
        list-claims
        list-repos (--org ORG)

Command:
  get - Gets the value of the sub customization
  set - Sets the value of the sub customization 
  use-default - Use default value instead of a customization
  list-claims - Lists the claims available for customization
  list-repos - Generates a tsv with all repo configurations. It has two fields: if repo uses default claims and the claims list.

Options:
    -h, --help                    : Show script help
    -o, --org                     : Name of the organization to get/set sub
    -r, --repo,--repository       : Name of the repo to get/set sub
    -q, --jq                      : JQ expression to filter the get output (optional)
    -s, --subs                    : The list of sub customization to set (comma separated). 
                                    Mandatory for organization set operation and optional for repo set operation.

Description:

Allows setting and getting OIDC sub customization claims for an organization or a repo.

Learn more on https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect

By default the output JSON returned by the API calls. You can use the --jq option to filter the output.

Examples:
  # Get the sub customization for the organization mona
  get --org mona
  # Sets the sub customization for the organization mona with claims repository_id,environment
  set --org mona --sub "repo,context,job_workflow_ref"
  # Gets the list of claims available for customization
  list-claims
  # get only the claims list
  get --org mona --jq ".include_claim_keys" 
  # Gets the settings for all repos in a org
  list-repos --org mona  
```

## Permissions

The following scopes are required to use this command:

- Organization:

  - `read:org` To get custom claims
  - `write:org` To set custom claims

- Repository:

  - `repo` To get and set custom claims

If you are unsure which scopes you have, you can get the list of scopes of the logged in user with one of the following commands:

```
$ gh auth status
github.com
  ✓ Logged in to github.com as XXXXXX  (/home/XXXXXXXX/.config/gh/hosts.yml)
  ✓ Git operations for github.com configured to use https protocol.
  ✓ Token: gho_************************************
  ✓ Token scopes: admin:org, delete_repo, gist, repo, workflow
```

```console
$ gh api user -i --silent | grep -i 'X-Oauth-Scopes:'
X-Oauth-Scopes: admin:org, delete_repo, gist, repo, workflow
```

To get more permissions you can use the [gh auth refresh](https://cli.github.com/manual/gh_auth_refresh) command with `--scopes` parameter.

## Debugging

To understand which subject claim is being sent to the OIDC provider from a GitHub action, you can run the following command in a workflow step:

```sh
  curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" "$ACTIONS_ID_TOKEN_REQUEST_URL" \
    -H "Accept: application/json; api-version=2.0" \
    -H "Content-Type: application/json" | cut -f2 -d . | base64 -d | jq -r .sub
```

A simpler alternative would be enabling debug logging since actions tend to emit more logs. You can enable debug logs by setting the `ACTIONS_STEP_DEBUG` environment variable to `true`.

For example the [azure/login](https://github.com/azure/login) action when running in debug mode will log the subject claim being sent to Azure.

This an example of `azure/login` logs snippet of repo with a customized claim `repo,ref` :
```text
Run Azure/login@v1.4.6
Using OIDC authentication...
Federated token details: 
 issuer - https://token.actions.githubusercontent.com 
 subject claim - repo:mona/repo:ref:refs/heads/main
```
