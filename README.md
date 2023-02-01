# README

This is a test app for trying to get Github Actions passing.

## Articles Referenced:

- https://docs.knapsackpro.com/2021/how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec
- https://dev.to/buildwithallan/how-to-set-up-a-ci-workflow-using-github-actions-4818
- https://boringrails.com/articles/building-a-rails-ci-pipeline-with-github-actions/

## Terminal Output Thus Far

```
1549  rails new github_rails_ci -T -d="postgresql" --skip-spring --skip-turbolinks
 1550  cd github_rails_ci
 1551  code .
 
 1561  git remote add origin git@github.com:arnaldoaparicio/github_rails_ci.git
 1562  git branch -M main
 1563  git push -u origin main
  1552  rails generate scaffold Widget name:string
 1553  rspec
 1554  rails db:setup
 1555  rails db:migrate
 1556  rspec
 1557  git add .
 1558  git commit -m "Install rspec and scaffold widget"
 1559  git push origin main
 1560  touch .github
 1561  touch .github
 1562  ls
 1563  gs
 1564  git status
 1565  rm .github
 1566  code .github/workflows/run_spec.yml
 1567  history

 
```