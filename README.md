# reusable

## `buildInfo`
This reusable workflow, BuildInfo, gathers and processes metadata such as pull 
request information, repository name, branch details, and git tags. It produces 
a set of outputs that can be used in downstream jobs, such as generating 
release notes or making decisions based on branch names and release tags.

##### <ins>Inputs<ins>
This workflow does not currently require any explicit inputs from the caller. 
It operates based on environment variables provided by GitHub Actions, such as 
pull request data and branch names.

##### <ins>Outputs<ins>
Here are the outputs that this workflow generates and makes available to the 
calling workflow:

- `PR_NUMBER`: number of pull request triggering the workflow (if applicable).
- `PR_AUTHOR`: github username of the author of the pull request.
- `REPO_NAME`: name of the repository.
- `CODE_OWNER`: first code owner listed in caller workflow  `.github/CODEOWNERS` 
  file. The assumption is the this used in a non-org repo with 1 code owner.
- `BRANCH_NAME`: name of the source branch for the workflow (supports both 
	pull requests and push events).
- `VERSION_NUMBER`: version extracted from the branch name; expected 
	format:release/vmajor.minor.patch or hotfix/vmajor.minor.patch).
- `RELEASE_TITLE`: release title, typically a combination of the repository name
	 and the latest release tag.
- `NEWEST_TAG`: most recently merged git tag.
- `NEWEST_TAG_PRESENT`: boolean indicating whether a newest tag is found (true) 
	or not (false).
- `OLDEST_TAG`: oldest merged git tag.
- `OLDEST_TAG_PRESENT`: boolean indicating whether an oldest tag is found (true)
	 or not (false).
- `LAST_RELEASE_TAG`: most recent release tag from GitHub.
- `LAST_RELEASE_TAG_PRESENT`: boolean indicating whether a last release tag is 
	found (true) or not (false).

##### <ins>Usage<ins>
To use this workflow in another workflow, you can reference it like this:
```yaml
jobs:
  build:
    uses: owner/repoName/.github/workflows/buildyaml@main
    with:
      # No explicit inputs needed
```
Then, in your caller workflow, you can access the outputs from the BuildInfo
 workflow using the needs context:
 ```
 - name: Use Build Info
  run: |
    echo "Branch Name: ${{ needs.build-info.outputs.BRANCH_NAME }}"
    echo "PR Number: ${{ needs.build-info.outputs.PR_NUMBER }}"
    echo "Newest Tag: ${{ needs.build-info.outputs.NEWEST_TAG }}"
 ```
##### <ins>Key Considerations</ins>
- __Branch Name Validation:__ This workflow expects branch names to follow the 
pattern release/vmajor.minor.patch or hotfix/vmajor.minor.patch. It will fail 
if this pattern is not matched.
- __GitHub Token:__ If you are using steps that interact with the GitHub API 
(e.g., gh release list), ensure that GITHUB_TOKEN is properly provided in the 
caller workflow’s environment.
- __CODEOWNERS File:__ Ensure your repository has a .github/CODEOWNERS file if 
you rely on the CODE_OWNER output.
- __Tag Management:__ This workflow fetches git tags, so make sure your tags are correctly set up and that git fetch --tags works as expected.