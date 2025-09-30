# Making Changes to CDA Website

## Overview

The production website is [built](https://github.com/CancerDataAggregator/internal_documentation/blob/main/CDA-HelpDesk/Infrastructure.md#build-process) from the main branch of the CDA-HelpDesk repo, and the main branch is protected so that changes can't be made to it directly. Every change must be made on some other branch, made into a pull request (PR) and then merged into main. This helps to ensure that changes are tested and the production site doesn't accidently break. The steps for making a change to production are:

1. Make a new branch from main
2. Make any edits you want in your new branch
3. Make a PR of your new branch to main
4. Wait for the tests to run on your PR
5. When the tests have completed, correct any spelling errors and check the status of the `docs/readthedocs.org:cda` test
   - If `docs/readthedocs.org:cda` fails, then your edits have caused the website to become unbuildable
   - If `docs/readthedocs.org:cda` passes, there will be a link to "view details". Click this link to view a rendered version of the website with your changes, and ensure this preview looks the way you intended


## 
