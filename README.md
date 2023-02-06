# README

This is a test app for trying to get Github Actions passing.

## Articles Referenced:

- https://docs.knapsackpro.com/2021/how-to-run-ruby-on-rails-tests-on-github-actions-using-rspec
- https://dev.to/buildwithallan/how-to-set-up-a-ci-workflow-using-github-actions-4818
- https://boringrails.com/articles/building-a-rails-ci-pipeline-with-github-actions/

## Terminal Output Thus Far

```
1549  rails _7.0.1_ new github_rails_ci -T -d="postgresql" --skip-spring --skip-turbolinks
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

## UPDATE 02/01/2023
Finally have tests running on Github Actions

In ```run_spec.yml``` i removed the postgres env except ```POSTGRES_PASSWORD```

All credentials are located further down the file. These need to be the same as the ones in ```config/database.yml```

config/database.yml, go to test section and add ```host```, ```username```, and ```password```.

then head to ```config/application.rb``` and uncomment ```require "rails/test_unit/railtie"```


# Draft Blog Post


## Introduction

## Instructions

### Step 1: 'rails new'

### Step 2: Ensure local and remote repositories are connected

### Step 3: Use 'rails generate scaffold' to generate tests 

For the purposes of this guide, you can auto-generate tests, but if you're trying to set up github actions on an existing rails project, you _likely_ already have tests, so skip this step.

```
rails g Widget name:string
```
### Step 4: Create .yml file that will be read by github actions

To trigger github actions, github will expect a yml file inside of `.github/workflows/`

```
code .github/workflows/run_tests.yml
```

### Step 5: Fill new yml file with instructions for github actions:

```yml
name: CI 
on: [push, pull_request] 
jobs:
  test:
    runs-on: ubuntu-latest
    services: 
      db:
        image: postgres:14
        ports: ['5432:5432']
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
    - uses: actions/checkout@v3
    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: '2.7.6'
        bundler-cache: true
    - uses: borales/actions-yarn@v2.0.0
      with:
        cmd: install
    - name: Build and run tests
      env:
        PG_DATABASE: postgres
        PG_HOST: localhost
        PG_USER: postgres
        PG_PASSWORD: postgres
        RAILS_ENV: test
        
      run: |
        sudo apt-get -yqq install libpq-dev
        gem install bundler
        bundle install --jobs 4 --retry 3
        bundle exec rails db:setup
        bundle exec rake test
        bundle exec rspec spec
```

## Step 7: Slow down and study the file 


## Step 8: Commit it and see if your tests work.



## Conclusion

Now, having read this guide, you know how to go from `rails new` in your terminal, to a successful github actions "run" than checks that all of your tests are passing.

Sally forth and prosper.