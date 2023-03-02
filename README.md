## _This repo will use to showcase the commitizen CI configuration_


Among the automated versioning tools, ‘commitizen’ came out as the most suitable option for our requirements since it is not limited to a specific technology stack like Java or React. It can be used globally for any project and can automate the process if the project is in a proper CI environment.

Here's how to integrate commitizen into the Github actions of any project. 

1. Create two files named .cz.yaml and Changelog  in the root directory of the project. 
2. Add the following content into the .cz.yaml file.
```sh
    ---
    commitizen:
      name: cz_conventional_commits
      tag_format: $version
      version: 3.0.0
```
3. Create a personal access token. Follow the instructions [here](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line#creating-a-token). And copy the generated key (you can use any among the fine-grained or classic personal access tokens).

4. Add the read-and-write access to the content permission of the token by clicking edit right next to the token.

5. Create a secret called PERSONAL_ACCESS_TOKEN, with the copied key, by going to your project repository and then Settings -> Secrets and variables -> New repository secret -> Add new secret.

6. In your project create a new file .github/workflows/bumpversion.yml and add the below content.
```sh
name: Bump version

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  bump-version:
    if: "!startsWith(github.event.head_commit.message, 'bump:')"    
    runs-on: ubuntu-latest
    name: "Bump version and create changelog with commitizen"    
    steps:
      - name: Check out
        uses: actions/checkout@v2
        with:
          token: "${{ secrets.PERSONAL_ACCESS_TOKEN }}"          
          fetch-depth: 0
      - name: Create bump and changelog
        uses: commitizen-tools/commitizen-action@master
        with:
          github_token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          changelog_increment_filename: body.md
      - name: Release
        uses: softprops/action-gh-release@v1
        with:
          body_path: "body.md"          
          tag_name: ${{ env.REVISION }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

8. Push this file and that’s it! 

