# Testing Rails with RSpec

Beginners introduction to testing Ruby On Rails application with RSpec and Capybara.


## Install Rails with RSpec (v0.1)


1. Create new rails project called `portfolio`: `rails new portfolio`;

2. Add `rspec` to `Gemfile`:

```ruby

  group :development, :test do
    gem 'rspec-rails', '~> 3.7'
  end

```

3. Install dependencies: `bundle install`;

4. Execute `bundle exec rails generate rspec:install` to create a `/spec` dir;

5. Run rspec: `bundle exec rspec`


## Your first spec (v0.2)


1. Create first spec:

```ruby

require "rails_helper"

RSpec.describe "hello spec" do
  describe "math" do
    expect(6 * 7).to eq(43)
  end
end

```

2. And catch an error:

```

Failures:

  1) hello spec math should be able to perform basic math
     Failure/Error: expect(6 * 7).to eq(43)
     
       expected: 43
            got: 42


```

3. Fix the error to get test passed:

```ruby

require "rails_helper"

RSpec.describe "hello spec" do
  describe "math" do
    it "should be able to perform basic math" do
      # expect(6 * 7).to eq(43) # => false
      expect(6 * 7).to eq(42)
    end
  end
end

```

4. Add another spec with empty string:

```ruby

require "rails_helper"

RSpec.describe "hello spec" do
  # ...
  describe String do
    let(:string) { String.new }
    it "should provide an empty string" do
      expect(string).to eq("")
    end
  end
end

```


## Create a unit test for Article model (v0.3)


1. Create scaffolding project `Project`:

```bash

bundle exec rails g scaffold project title:string description:text

RAILS_ENV=test bundle exec rake db:migrate

```

2. Create `spec/models/article_spec.rb`:

```ruby

require "rails_helper"

RSpec.describe Project, type: :model do
  context "validations tests" do
    it "ensures the title is present" do
      article = Project.new(description: "Content of the description")
      expect(project.valid?).to eq(false)
    end

    it "ensures the body is present" do
      article = Project.new(title: "Title")
      expect(project.valid?).to eq(false)
    end
    
    it "should be able to save article" do
      article = Project.new(title: "Title", description: "Some description content goes here")
      expect(project.save).to eq(true)
    end
  end

  context "scopes tests" do

  end
end

```

3. Add some presence validators to `app/models/project.rb`:

```ruby

class Project < ApplicationRecord
  validates_presence_of :title, :description
end

```

4. Add scope specs:

```ruby

require "rails_helper"

RSpec.describe Project, type: :model do
  # ...

  context "scopes tests" do
    let(:params) { { title: "Title", decription: "some description" } }
    before(:each) do
      Project.create(params)
      Project.create(params)
      Project.create(params)
    end

    it "should return all projects" do
      expect(Article.active.count).to eq(3)
    end

  end
end

```


##  Create functional test for Articles controller (v0.4)

1. Create Articles spec:

```ruby

require "rails_helper"

RSpec.describe ProjectsController, type: :controller do
  context "GET #index" do
    it "returns a success response" do
      get :index
      # expect(response.success).to eq(true)
      expect(response).to be_success
    end
  end

  context "GET #show" do
    let!(:project) { Project.create(title: "Test title", description: "Test description") }
    it "returns a success response" do
      get :show, params: { id: article }
      expect(response).to be_success
    end
  end
end

```

2. Add `--format documentation` to `.rspec`


## Create integration spec and a home page (v0.5)


1. Add Capybara to `Gemfile`:

```ruby

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'rspec-rails', '~> 3.7'
  gem 'capybara'
end

```

2. Add these lines to your `spec/rails_helper.rb` file:

```ruby

require 'capybara/rails'
require 'capybara/rspec'

```

3. Create first feature test by running `bundle exec rails g rspec:feature home_page`:

```ruby

require "rails_helper"

RSpec.feature "Visiting the homepage", type: :feature do
  scenario "The visitor should see projects" do
    visit root_path
    expect(page).to have_text("Projects")
  end
end

```

4. Add `root` to `routes.rb`:

```ruby

Rails.application.routes.draw do
  root "projects#index"
  resources :projects
end

```

## Create integration spec for Projects


1. `bundle exec rails g rspec:feature projects`

2. Full Projects test:

```ruby

require 'rails_helper'

RSpec.feature "Projects", type: :feature do
  context "Create new project" do
    before(:each) do
      visit new_project_path
      within("form") do
        fill_in "Title", with: "Test title"
      end
    end

    scenario "should be successful" do
      fill_in "Description", with: "Test description"
      click_button "Create Project"
      expect(page).to have_content("Project was successfully created")
    end

    scenario "should fail" do
      click_button "Create Project"
      expect(page).to have_content("Description can't be blank")
    end
  end

  context "Update project" do
    let(:article) { Project.create(title: "Test title", description: "Test content") }
    before(:each) do
      visit edit_project_path(article)
    end

    scenario "should be successful" do
      within("form") do
        fill_in "Description", with: "New description content"
      end
      click_button "Update Project"
      expect(page).to have_content("Project was successfully updated")
    end

    scenario "should fail" do
      within("form") do
        fill_in "Description", with: ""
      end
      click_button "Update Project"
      expect(page).to have_content("Description can't be blank")
    end
  end

  context "Remove existing project" do
    let!(:article) { Project.create(title: "Test title", description: "Test content") }
    scenario "remove project" do
      visit projects_path
      click_link "Destroy"
      expect(page).to have_content("Project was successfully destroyed")
      expect(Project.count).to eq(0)
    end
  end
end

```


## Add simplecov gem:


1. Rails coverage report: `bundle exec rails stats`

2. To install codecov add to your `Gemfile`:

```ruby

gem 'simplecov', require: false, group: :test

```

3. Add to your `spec/rails_helper.rb`:

```ruby

require 'simplecov'
SimpleCov.start

```

4. Don't forget to add `coverage/` dir to your `.gitignore` file;


## Helpful links:


* https://relishapp.com/rspec/rspec-rails/docs

* https://github.com/teamcapybara/capybara

* https://github.com/colszowka/simplecov

* http://www.betterspecs.org/ 

* https://github.com/thoughtbot/factory_bot

* https://github.com/ffaker/ffaker
