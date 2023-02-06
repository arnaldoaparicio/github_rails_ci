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

To start from scratch, generate a new rails app on your local machine. This is the configuration I am using for this walkthrough, making sure to set `postgres` as the database, to make the github actions more real-world and realistic. 

```
$ rails _5.2.8.1_ new github_rails_ci -T -d="postgresql" --skip-spring --skip-turbolinks
# there will be lots of terminal output after running this command, but when it finishes:

$ cd github_actions_walkthrough
$ git add .
$ git commit -m "initial commit"
```

### Step 2: Ensure local and remote repositories are connected

If you already have an existing repository on github, skip this step. Otherwise we will create a new repo.

- Create new repository attached to your github account at https://github.com/new
- Add repository name. Ensure that it matches the name of your rails app (for example, I will use `github_actions_walkthrough`)
- Run github's prescribed terminal commands. 
- Refresh browser, make sure you can see your initial commit

```
# after creating repository on https://github.com/new:

$ git remote add origin git@github.com:arnaldoaparicio/github_actions_walkthrough.git
$ git branch -M main
$ git status
$ git push -u origin main

# refresh browser to verify results
```


### Step 3.0 : Install `rspec-rails` 

For the purposes of this guide, you can auto-generate tests, but if you're trying to set up github actions on an existing rails project, you _likely_ already have tests, so skip this step.

Add the `rspec-rails` gem to your `:development, :test` group in `Gemfile`

```
$ bundle install
$ rails generate rspec:install
```

### Step 3.1 : Use 'rails generate scaffold' to generate tests 


```

rails generate scaffold Widget name:string

```

It could be helpful to add and commit work so far, but that's up to you.

### Step 4: Create .yml file that will be read by github actions

To trigger github actions, github will expect a yml file inside of `.github/workflows/`

```
code .github/workflows/run_tests.yml
```

### Step 5: Fill new yml file with instructions for github actions:

```yml
# .github/workflows/run_spec.yml
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
        bundle exec rspec spec
```

# Step 6: Make sure your tests work locally

```

$ rails db:setup
$ rails db:migrate
$ rspec
```

## Step 7: Commit it and see if your tests work

Push your new `yml` file to Github, and check the `actions` tab on the repository to watch the action get run. For me, I'd head to http://github.com/arnaldoaparicio/github_actions_walkthrough/actions/


## Step 8: Realize your tests don't work because `config/database.yml` needs to be updated

The values in the `test` development block in `config/database.yml` need to include postgres user information to be used during the github Actions run. 

The values for `host`, `username`, and `password` should match what you have in `run_spec.yml`

```yml
# .github/workflows/run_spec.yml:28
 env:
    PG_DATABASE: postgres
    PG_HOST: localhost
    PG_USER: postgres
    PG_PASSWORD: postgres
    RAILS_ENV: test
```

```diff
diff --git a/config/database.yml b/config/database.yml
index 7555321..908e314 100644
--- a/config/database.yml
+++ b/config/database.yml
@@ -58,6 +58,10 @@ development:
 test:
   <<: *default
   database: github_actions_walkthrough_test
+  host: localhost
+  username: postgres
+  password: postgres

```

Push this all to Github, check the `actions` tab output, and you should see a green check, and if you drill into the `build and run tests` dropdown/collapsible menu item, you'll see some output like:

```
  13) /widgets DELETE /destroy destroys the requested widget
     # Add a hash of attributes valid for your model
     # ./spec/requests/widgets_spec.rb:118

  14) /widgets DELETE /destroy redirects to the widgets list
     # Add a hash of attributes valid for your model
     # ./spec/requests/widgets_spec.rb:125

Finished in 0.58442 seconds (files took 0.72949 seconds to load)
27 examples, 0 failures, 14 pending
```

Congrats!

## Conclusion

Now, having read this guide, you know how to go from `rails new` in your terminal, to a successful github actions "run" than checks that all of your tests are passing.

Sally forth and prosper.