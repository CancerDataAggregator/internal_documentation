# Making Changes to CDA Website

## Overview

The production website is [built](https://github.com/CancerDataAggregator/internal_documentation/blob/main/CDA-HelpDesk/Infrastructure.md#build-process) from the main branch of the CDA-HelpDesk repo, and the main branch is protected so that changes can't be made to it directly. Every change must be made on some other branch, made into a pull request (PR) and then merged into main. This helps to ensure that changes are tested and the production site doesn't accidently break. 

## Quick Start

The steps for making a change to production are:

1. Make a new branch from main
2. Make any edits you want in your new branch
3. Make a PR of your new branch to main
4. Wait for the tests to run on your PR
5. When the tests have completed, correct any spelling errors and check the status of the `docs/readthedocs.org:cda` test
   - If `docs/readthedocs.org:cda` fails, then your edits have caused the website to become unbuildable
   - If `docs/readthedocs.org:cda` passes, there will be a link under `...` to "view details". Click this link to view a rendered version of the website with your changes, and ensure this preview looks the way you intended
6. Make any additional edits on your same new branch and view the preview site until it looks the way you want
7. Assign someone to review your PR
8. Once the PR has one approval, you can merge it to main. Assuming you followed the previous steps and ensured the website rendered properly in preview, merging to main should trigger readthedocs to automatically rebuild and release the site with your changes.

>[!TIP]
>If your changes do not appear:
> - wait a little longer, the website is actually running quite a bit of python code and it can take up to 30 minutes to build
> - try clearing your cache, chrome especially will not always refresh pages properly
> - check the readthedocs project for errors and fix them

### Make edits, on GitHub step by step with pictures

1. Make a new branch from main
   i. Click on the button that says `main` on the homepage, and type in a name for your new branch
   <img width="677" height="566" alt="image" src="https://github.com/user-attachments/assets/3346034b-6cb2-4281-98d2-5417e9e11ea4" />
   
   ii. Click `Create branch xxxxxx from main`
   
   iii. You should now be on your new branch (the button no longer says main, it says your new branch name)
   <img width="437" height="231" alt="image" src="https://github.com/user-attachments/assets/90ca7e23-068d-4e5b-a24e-9ab00b9f472c" />

2. Make any edits you want in your new branch
   
   i. Click on any file or folder to navigate to the file you want to change

   ii. Click the pencil icon on your selected file
   <img width="2440" height="670" alt="image" src="https://github.com/user-attachments/assets/ed6d3520-329d-4ff4-a616-5cc7f1c29341" />

   iii. Make your edits. Be sure to use the correct language for the file you are editing:
        - `.md` files are in [markdown](https://www.markdownguide.org/)
        - `.yml` files are in [yaml](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started)
        - `.html` filea are in [HTML](https://www.w3schools.com/html/html_intro.asp)
        - `.ipynb` files are [jupyter notebooks](https://ipython.org/notebook.html)
>[!TIP]
> `.md`, `.yml`, `.html` are all various flavors of text files. You can edit them easily in your browser. Jupyter notebooks *render* nicely in github, but are nearly impossible to *edit* in github. To edit them, you'll want to clone the repo to your local machine and [work locally].   
   
>[!NOTE]
> The markdown files can actually be a mix of all the text languages. For instance, nearly every page has a yaml header that adds extra info, and some have sections with html because they needed formatting that is not available in the markdown language. Similarly, some jupyter notebooks have a bit of html to make nice call outs or fix formatting issues.

   iv. Save (commit) your edits by clicking 'commit changes'
   
<img width="368" height="205" alt="image" src="https://github.com/user-attachments/assets/b4a817a9-e6c8-44eb-9366-6aa85169d7ab" />
   
   v. and then fill in the title and/or description of your change and click 'commit changes' in the pop up box - be sure to commit to your branch
<img width="508" height="595" alt="image" src="https://github.com/user-attachments/assets/1cb8bc1e-16ee-42e7-91b4-6926c02a1725" />

3. Make a PR of your new branch to main

   i. Navigate to the main page of the repo, where you should find a notification about your changes
   <img width="1088" height="329" alt="image" src="https://github.com/user-attachments/assets/85483ed0-2988-4664-9171-68d58251e281" />

   ii. Click 'Compare & pull request'

   iii. This will take you to the pull request form. It should be autofilled with your comments from your change commits.

   iv. Check that the PR is in the correct direction, which is your new branch _into_ main (note the direction of the arrow):
   <img width="463" height="165" alt="image" src="https://github.com/user-attachments/assets/de4240dd-abf1-4d66-b242-1c0d9f215453" />

   v. Add any labels, extra notes, who you would like to review, etc as desired and click 'Create pull request'
   <img width="1324" height="707" alt="image" src="https://github.com/user-attachments/assets/28994f10-1017-49d3-8cc0-e686c6824da3" />


4. Wait for the tests to run on your PR
   
   i. GitHub will immediatly start running our standard tests (currently spellcheck and render check)
   <img width="1324" height="707" alt="image" src="https://github.com/user-attachments/assets/57739961-fd16-49ca-a7c8-cc85caee0cc9" />

5. When the tests have completed, correct any spelling errors and check the status of the `docs/readthedocs.org:cda` test

   i. The spelling report always fails, because it always finds a few false errors, such as BingXings name. Ignore those and fix any real ones

   ii. A passing render check, `docs/readthedocs.org:cda` will have a green checkmark:
   <img width="882" height="491" alt="image" src="https://github.com/user-attachments/assets/5b252da5-db9c-40d8-abab-b9a4089620a6" />

       a. click the `...` then "View details"
      <img width="882" height="385" alt="image" src="https://github.com/user-attachments/assets/9a34c20e-9198-4409-94b2-21f704de47d8" />

       b. this will open a preview version of the website that is clearly labeled as being from a pull request rather than main:
   <img width="2900" height="750" alt="image" src="https://github.com/user-attachments/assets/740f6e00-86bc-4267-a79c-269f50639327" />
       - navigate as you would the production site to check your changes
>[TIP]
>You can have as many simultaneous preview sites as you want. For example, if you want to try two different styles for a page and compare them side by side. To do that, make two new branches, give each one a different style, then make a PR of each of them. PRs are numbered, and previews are numbered to match so you can tell them apart.

   iii. A failing render check, `docs/readthedocs.org:cda` says it failed:

   <img width="882" height="107" alt="image" src="https://github.com/user-attachments/assets/24616f29-9b95-4302-84e1-41d4c85acce3" />

        a. click the `...` then "View details"
         <img width="882" height="385" alt="image" src="https://github.com/user-attachments/assets/9a34c20e-9198-4409-94b2-21f704de47d8" />

        b.  This will open the readthedocs build page for this PR. You may need to sign in to readthedocs to see it. Sign in using your GitHub account.
        <img width="1450" height="912" alt="image" src="https://github.com/user-attachments/assets/5ab5594f-da4b-47f4-add1-12cf3bcca9c4" />

        c. Generally looking at the "Raw log" or "Debug" info will point you towards the error. Typical errors are due to jupyter notebook code failing, incorrect changes to config files, bad markdown/yaml/html. Usually the file that caused the problem is listed.


6. Make any additional edits on your same new branch and re-view the preview site until it looks the way you want

7. Assign someone to review your PR
   i. click the gear icon next to "Reviewers" and choose one or more people from the list
   <img width="414" height="489" alt="image" src="https://github.com/user-attachments/assets/625cc8d8-6a48-4ccd-91b2-cd85e0ca01e3" />

8. Once the PR has one approval, you can merge it to main.

### Make edits locally

1. In a terminal run the command `git clone git@github.com:CancerDataAggregator/CDA-HelpDesk.git` or `git clone https://github.com/CancerDataAggregator/CDA-HelpDesk.git` depending on how you set up your github authorization
2. `cd` into the new CDA-HelpDesk folder
3. create a new branch `git branch <new-branch-name>`
4. Make changes using your favorite editor(s). If you don't have a favorite, [VS code](https://code.visualstudio.com/) can easily display and make editable all file types in this repo
5. Save your changes: `git commit -a -m <message text>
6. Push your changes to github `git push` (likely github will complain and give you a more complex command to copy/paste in)
7. Go to step 3 of [the previous section](https://github.com/CancerDataAggregator/internal_documentation/edit/main/CDA-HelpDesk/MakingChanges.md#make-edits-on-github-step-by-step-with-pictures) and continue on from there
