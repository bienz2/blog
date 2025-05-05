---
title: "Automatically Compile and Deploy Latex Documents"
date: 2025-03-28
categories: [website, automation]
---

The steps below allow you to automatically push all Latex updates to publicly available PDFs.  I use this for all class assignments so that I can update any errors and make clarifications in real-time for the students.  It is also very useful for things like keeping your CV up-to-date on your website.

1. Open your overleaf document, click Menu -> GitHub to create a corresponding GitHub repository.
2. Go to your repository on Github.com
3. Create a new workflow for your project:
    a. Click Add file + Create new file
    b. Name the file ".github/workflows/publish.yml"
4. Give the workflow a name:
```
name: Compile and Deploy
```
5. Decide when you want new PDFs to be complied.  For instance, I only want to re-run the latex compilation when a new file is pushed to my Homework folder, and only on the main branch:
```
on:
  push:
    branches: 
      - main
    paths:
      - 'Homework/**'

  workflow_dispatch:```
6.You need to give the workflow the appropriate permissions for the PDF to be deployed to github pages:
```
permissions:
  contents: write
  pages: write
  id-token: write
```
7. Whenever the workflow runs, you need to gain access to latex.  While you can compile latex from scratch, this is time consuming.  I highly recommend using the existing docker container `xu-cheng/latex-action@v3`.  You need to pass the root file (such as main.tex) that you want to compile and deploy.  I want to compile all written and coding assignments, so I have listed a number of files:
```
jobs:
  build_latex:
    runs-on: ubuntu-latest
    environment:
      name: github-pages

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Compile LaTeX documents
        uses: xu-cheng/latex-action@v3
        with:
          root_file: |
            Homework/code1.tex
            Homework/code2.tex
            Homework/code3.tex
            Homework/code4.tex
            Homework/code5.tex
            Homework/hw1.tex
            Homework/hw2.tex
            Homework/hw3.tex
            Homework/hw4.tex
            Homework/hw5.tex
          latexmk_use_xelatex: false
```
8. I want to move the compiled PDFs into a folder called `files` so that I can deploy all PDFs within a single folder:
```
      - name: Move PDFs to files folder
        run: |
          mkdir -p files/
          mv *.pdf files/
```

9. Finally, I want to setup GitHub pages, upload the PDFs, and deploy the pages:
```
      - name: Setup GitHub Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: files

      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```
10. After compiling, you won't automatically see the PDFs.  You need to go to your GitHub repository Settings, click on `Pages` on the left hand side, and select `Deploy from a branch` under Source.  Then, select the branch as gh-pages.

11. Finally, your PDFs are live.  You can view them at https://<orgname>.github.io/<reponame>/<filename>.pdf.  For example, my file `code1.tex` is available at https://ProfessorBienz.github.io/CS481/code1.pdf.  (Note, this reroutes you to teaching.amandabienz.com/CS481/code1.pdf because I have rerouted ProfessorBienz.github.io to my own custom domain.  For more information on this process, check out the blog post TODO).

The full workflow for my CS481 repository is available here: 
