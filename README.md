# The RSpec Style Guide

This RSpec style guide outlines our recommended best practices so that our
developers can write code that can be maintained by other (future) developers. 
This is meant to be a style guide that reflects real-world usage, as well as a 
guide that holds to an ideal that has been agreed upon by many of the people it 
was intended to be used by.

The guide is separated into sections based on the different pieces of an entire 
spec file. There was an attempt to omit all obvious information, if anything is
unclear, feel free to open an issue asking for further clarity.

Per the comment above, this guide is a work in progress - some rules are simply
lacking thorough examples, but some things in the RSpec world change week by 
week or month by month. With that said, as the standard changes this guide is 
meant to be able to change with it.

* Do not leave line returns after `context` or `describe` descriptions. It does
  not make the code any easier to read and lowers the value of logical chunks.

    ```Ruby
    # bad
    describe Article do

      describe "#summary" do

        context "when there is a summary" do

          it "returns the summary" do
            # ...
          end
        end
      end
    end

    # good
    describe Article do
      describe "#summary" do
        context "when there is a summary" do
          it "returns the summary" do
            # ...
          end
        end
      end
    end
    ```

* Leave one line return after `let`, `subject`, and `before`/`after` blocks.

    ```Ruby
    # bad
    describe Article do
      subject { FactoryGirl.create(:some_article) }
      describe "#summary" do
        # ...
      end
    end

    # good
    describe Article do
      subject { FactoryGirl.create(:some_article) }

      describe "#summary" do
        # ...
      end
    end
    ```

* Only group `let`, `subject` blocks and separate them from `before`/`after` 
  blocks. It makes the code much more readable.

    ```Ruby
    # bad
    describe Article do
      subject { FactoryGirl.create(:some_article) }
      let(:user) { FactoryGirl.create(:user) }
      before do
        # ...
      end
      after do
        # ...
      end
      describe "#summary" do
        # ...
      end
    end

    # good
    describe Article do
      subject { FactoryGirl.create(:some_article) }
      let(:user) { FactoryGirl.create(:user) }

      before do
        # ...
      end

      after do
        # ...
      end

      describe "#summary" do
        # ...
      end
    end
    ```

* Leave one line return around `it` blocks. This helps to separate the 
  expectations from their conditional logic (contexts for instance).

    ```Ruby
    # bad
    describe "#summary" do
      let(:item) { mock('something') }

      it "returns the summary" do
        # ...
      end
      it "does something else" do
        # ...
      end
      it "does another thing" do
        # ...
      end
    end

    # good
    describe "#summary" do
      let(:item) { mock('something') }

      it "returns the summary" do
        # ...
      end

      it "does something else" do
        # ...
      end
      
      it "does another thing" do
        # ...
      end
    end
    ```

* There is no need to specify `(:each)` for `before`/`after` blocks, as it is
  the default functionality. There are almost zero cases to use `(:all)` 
  anymore, but if you do find one - just write it out like `before(:all)`

    ```Ruby
    # bad
    describe "#summary" do
      before(:each) do
        subject.summary = "something"
      end
    end

    # good
    describe "#summary" do
      before do
        subject.summary = "something"
      end
    end
    ```

* Do not write "should" in the beginning of your `it` blocks. The descriptions 
  represent actual functionality - not what might be happening.
    
    ```Ruby
    # bad
    it "should return the summary" do
      # ...
    end
    
    # good
    it "returns the summary" do
      # ...
    end
    ```

* Use just one expectation per example.

    ```Ruby
    # bad
    describe ArticlesController do
      #...

      describe 'GET new' do
        it 'assigns new article and renders the new article template' do
          get :new
          assigns[:article].should be_a_new Article
          response.should render_template :new
        end
      end

      # ...
    end

    # good
    describe ArticlesController do
      #...

      describe 'GET new' do
        it 'assigns a new article' do
          get :new
          assigns[:article].should be_a_new Article
        end

        it 'renders the new article template' do
          get :new
          response.should render_template :new
        end
      end

    end
    ```

* `context` blocks should pretty much always have an opposite negative case. It
  should actually be a strong code smell if there is a single context (without 
  a matching negative case) that it needs refactoring, or may have no purpose.

  ```Ruby
    # bad

    # This is a case where refactoring is the correct choice
    describe "#attributes" do
      context "the returned hash" do 
        it "includes the display name" do
          # ...
        end

        it "includes the creation time" do
          # ...
        end
      end
    end

    # This is a case where the negative case needs to be tested, but wasn't
    describe "#attributes" do
      context "when display name is present" do
        before do
          subject.display_name = "something"
        end

        it "includes the display name" do
          # ...
        end
      end
    end

    # good

    # Refactored
    describe "#attributes" do
      subject { FactoryGirl.create(:article) }

      its(:attributes) { should include subject.display_name }
      its(:attributes) { should include subject.created_at }
    end

    # Added the negative case
    describe "#attributes" do
      context "when display name is present" do
        before do
          subject.display_name = "something"
        end
        
        it "includes the display name" do
          # ...
        end
      end

      context "when display name is not present" do
        before do
          subject.display_name = nil
        end

        it "does not include the display name" do
          # ...
        end
      end
    end
  ```

* `context` block descriptions should always start with "when"

    ```Ruby
    # bad
    context "the display name is not present" do
      # ...
    end

    # good
    context "when the display name is not present" do
      # ...
    end
    ```

* `it` block descriptions should never end with a conditional. This is a code
  smell that the `it` most likely needs to be wrapped in a `context`.

    ```Ruby
    # bad
    it "returns the display name if it is present" do
      # ...
    end

    # good
    context "when display name is present" do
      it "returns the display name"
    end

    # This encourages the addition of negative test cases that might have
    # been overlooked
    context "when display name is not present" do
      it "returns nil"
    end
    ```

* Name the `describe` blocks as follows:
  * use hash "#method" for instance methods
  * use dot ".method" for class methods

  Given the following exists
    ```Ruby
    class Article
      def summary
        #...
      end

      def self.latest
        #...
      end
    end
    ```

    ```Ruby
    # bad
    describe Article do
      describe 'summary' do
        #...
      end

      describe 'latest' do
        #...
      end
    end

    # good
    describe Article do
      describe '#summary' do
        #...
      end

      describe '.latest' do
        #...
      end
    end
    ```

* Do not write iterators to generate tests. When another developer adds a 
  feature to one of the items in the iteration, he must then break it out into
  a separate test - he is forced to edit code that has nothing to do with his
  pull request.

  ```Ruby
    # bad
    [:new, :show, :index].each do |action|
      "it returns 200" do
        get action
        response.should be_ok
      end
    end

    # good (more verbose for the time being, but better for the future development)
    describe "GET new" do
      it "returns 200" do
        get :new
        response.should be_ok
      end
    end

    describe "GET show" do
      it "returns 200" do
        get :show
        response.should be_ok
      end
    end

    describe "GET index" do
      it "returns 200" do
        get :index
        response.should be_ok
      end
    end
  ```

* Use [Factory Girl](https://github.com/thoughtbot/factory_girl) to create test
  objects. You should very rarely have to use `ModelName.create` within a spec.

  ```Ruby
  subject { FactoryGirl.create(:some_article) }
  ```

* Use mocks and stubs with caution. While they help to improve the performance 
  of the test suite, you can mock/stub yourself into a false-positive state very
  easily. When resorting to mocking and stubbing, only mock against a small, 
  stable, obvious (or documented) API, so stubs are likely to represent reality 
  after future refactoring.

  * [joahking](https://github.com/joahking) gives a good explanation of when to 
  use stubs/mocks:
    * Performance: To prevent running a slow, unrelated task.
    * Determinism: To ensure the test gives the same result each
      time. e.g. Kernel#rand, external web services.
    * Vendoring: When relying on 3rd party code used as a "black box",
      which wasn't written with testability in mind.
    * Legacy: Stubbing old code that requires complex setup. (New code
      should not require complex setup!)
    * BDD: To remove the dependence on code that does not yet exist.
    * Controller / Functional tests:
    > In a controller spec, we don't care about how our data objects are created or what data they contain; we are writing expectations for the functional behavior of that controller, and that controller only. Mocks and stubs are used to decouple from the model layer and stay focused on the task of specing the controller.

    ```Ruby
    # mocking a model
    article = mock_model(Article)

    # stubbing a method
    Article.stub(:find).with(article.id).and_return(article)
    ```

  NOTE: if you stub a method that could give a false-positive test result, you 
  have gone too far. See below:

    ```Ruby
    # bad (stubbing too far)
    subject { mock_model(Article) }

    describe "#summary" do
      context "when summary is not present" do
        # This stub_chain is stubbing the #nil? method, which makes the
        # test pass, but you are no longer testing the functionality of 
        # the code, you are testing the functionality of the test suite.
        # This test would pass if there was not a single line of code 
        # written for the Article class.
        subject.stub_chain(:summary, :nil?).and_return(true)

        it "returns nil" do
          subject.summary.should be_nil
        end
      end
    end


    # good (stubbing only what's necessary)
    subject { mock_model(Article) }

    describe "#summary" do
      context "when summary is not present" do
        # This is no longer stubbing all of the functionality, and will 
        # actually test the objects handling of the methods return value.
        subject.stub(:summary).and_return(nil)

        it "returns nil" do
          subject.summary.should be_nil
        end
      end
    end
    ```

* Always use [Timecop](https://github.com/travisjeffery/timecop) instead of stubbing anything on Time or Date.

    ```Ruby
    # bad
    it "offsets the time 2 days into the future" do
      current_time = Time.now
      Time.stub(:now).and_return(current_time)
      subject.get_offset_time.should be_the_same_time_as (current_time + 2.days)
    end

    # good
    it "offsets the time 2 days into the future" do
      Timecop.freeze(Time.now) do
        subject.get_offset_time.should be_the_same_time_as 2.days.from_now
      end
    end
    ```

    NOTE: `#be_the_same_time_as` is a RSpec matcher we added to the platform, it
    is not normally available to RSpec.

* Use `let` blocks instead of `before(:each)` blocks to create data for the spec
  examples. `let` blocks get lazily evaluated. It also removes the instance 
  variables from the test suite (which don't look as nice as local variables).

    ```Ruby
    # use this:
    let(:article) { FactoryGirl.create(:article) }

    # ... instead of this:
    before { @article = FactoryGirl.create(:article) }
    ```

* Use `let!` blocks when you want the content to be evaluated immediately (skip 
  the lazy loading altogether). 

  ```Ruby
  let!(:article) { FactoryGirl.create(:article) }
  ```

  NOTE: mostly, you should be using `let` (without the bang `!`) - but there are
  a few special cases when lazy loading can cause problems - this is the only 
  reason to use `let!`.

* Use `subject` when possible

    ```Ruby
    describe Article do
      subject { FactoryGirl.create(:article) }

      it 'is not published on creation' do
        subject.should_not be_published
      end
    end
    ```

* Use RSpec's "magical matcher" methods when possible. For instance, a class
  with the method `published?` should be tested with the following:

    ```Ruby
    it 'is published' do
      # actually tests subject.published? == true
      subject.should be_published
    end
    ```

* Use `its` when possible

    ```Ruby
    # bad
    describe Article do
      subject { FactoryGirl.create(:article) }

      it 'has the current date as creation date' do
        subject.creation_date.should == Date.today
      end
    end

    # good
    describe Article do
      subject { FactoryGirl.create(:article) }
      its(:creation_date) { should == Date.today }
    end
    ```
* Avoid incidental state as much as possible.

    ```Ruby
    # bad
    it "publishes the article" do
      article.publish
      
      # Creating another shared Article test object above would cause this
      # test to break
      Article.count.should == 2
    end

    # good
    it "publishes the article" do
      -> { article.publish }.should change(Article, :count).by(1)
    end
    ```

* Be careful not to focus on being "DRY" by moving repeated expectations into a 
  shared environment too early, as this can lead to brittle tests that rely too 
  much on one other.

### Views

* The directory structure of the view specs `spec/views` matches the
  one in `app/views`. For example the specs for the views in
  `app/views/users` are placed in `spec/views/users`.
* The naming convention for the view specs is adding `_spec.rb` to the
  view name, for example the view `_form.html.erb` has a
  corresponding spec `_form.html.erb_spec.rb`.
* `spec_helper.rb` need to be required in each view spec file.
* The outer `describe` block uses the path to the view without the
  `app/views` part. This is used by the `render` method when it is
  called without arguments.

    ```Ruby
    # spec/views/articles/new.html.erb_spec.rb
    require 'spec_helper'

    describe 'articles/new.html.erb' do
      # ...
    end
    ```

* Always mock the models in the view specs. The purpose of the view is
  only to display information.
* The method `assign` supplies the instance variables which the view
  uses and are supplied by the controller.

    ```Ruby
    # spec/views/articles/edit.html.erb_spec.rb
    describe 'articles/edit.html.erb' do
      it 'renders the form for a new article creation' do
        assign(:article, mock_model(Article).as_new_record.as_null_object)
        render
        rendered.should have_selector('form',
          method: 'post',
          action: articles_path
        ) do |form|
          form.should have_selector('input', type: 'submit')
        end
      end
    end
    ```

* Prefer the capybara negative selectors over should_not with the positive.

    ```Ruby
    # bad
    page.should_not have_selector('input', type: 'submit')
    page.should_not have_xpath('tr')

    # good
    page.should have_no_selector('input', type: 'submit')
    page.should have_no_xpath('tr')
    ```

* When a view uses helper methods, these methods need to be stubbed. Stubbing 
  the helper methods is done on the `template` object:

    ```Ruby
    # app/helpers/articles_helper.rb
    class ArticlesHelper
      def formatted_date(date)
        # ...
      end
    end
    ```

    ```Rails
    # app/views/articles/show.html.erb
    <%= "Published at: #{formatted_date(@article.published_at)}" %>
    ```
    
    ```Ruby
    # spec/views/articles/show.html.erb_spec.rb
    describe 'articles/show.html.erb' do
      it 'displays the formatted date of article publishing' do
        article = mock_model(Article, published_at: Date.new(2012, 01, 01))
        assign(:article, article)

        template.stub(:formatted_date).with(article.published_at).and_return('01.01.2012')

        render
        rendered.should have_content('Published at: 01.01.2012')
      end
    end
    ```

* The helpers specs are separated from the view specs in the `spec/helpers` 
  directory.

### Controllers

* Mock the models and stub their methods. Testing the controller should not 
  depend on the model creation.
* Test only the behaviour the controller should be responsible about:
  * Execution of particular methods
  * Data returned from the action - assigns, etc.
  * Result from the action - template render, redirect, etc.

        ```Ruby
        # Example of a commonly used controller spec
        # spec/controllers/articles_controller_spec.rb
        # We are interested only in the actions the controller should perform
        # So we are mocking the model creation and stubbing its methods
        # And we concentrate only on the things the controller should do

        describe ArticlesController do
          # The model will be used in the specs for all methods of the controller
          let(:article) { mock_model(Article) }

          describe 'POST create' do
            before { Article.stub(:new).and_return(article) }

            it 'creates a new article with the given attributes' do
              Article.should_receive(:new).with(title: 'The New Article Title').and_return(article)
              post :create, message: { title: 'The New Article Title' }
            end

            it 'saves the article' do
              article.should_receive(:save)
              post :create
            end

            it 'redirects to the Articles index' do
              article.stub(:save)
              post :create
              response.should redirect_to(action: 'index')
            end
          end
        end
        ```

* Use context when the controller action has different behaviour depending on 
  the received params.

    ```Ruby
    # A classic example for use of contexts in a controller spec is creation or update when the object saves successfully or not.

    describe ArticlesController do
      let(:article) { mock_model(Article) }

      describe 'POST create' do
        before { Article.stub(:new).and_return(article) }

        it 'creates a new article with the given attributes' do
          Article.should_receive(:new).with(title: 'The New Article Title').and_return(article)
          post :create, article: { title: 'The New Article Title' }
        end

        it 'saves the article' do
          article.should_receive(:save)
          post :create
        end

        context 'when the article saves successfully' do
          before { article.stub(:save).and_return(true) }

          it 'sets a flash[:notice] message' do
            post :create
            flash[:notice].should eq('The article was saved successfully.')
          end

          it 'redirects to the Articles index' do
            post :create
            response.should redirect_to(action: 'index')
          end
        end

        context 'when the article fails to save' do
          before { article.stub(:save).and_return(false) }

          it 'assigns @article' do
            post :create
            assigns[:article].should be_eql(article)
          end

          it 're-renders the "new" template' do
            post :create
            response.should render_template('new')
          end
        end
      end
    end
    ```

### Models

* Do not mock the models in their own specs.
* Use `FactoryGirl.create` to make real objects, or just use a new (unsaved) 
  instance with `subject`.

    ```Ruby
    describe Article do
      let(:article) { FactoryGirl.create(:article) }

      # Currently, "subject" is the same as "Article.new"
      it "is an instance of Article" do
        subject.should be_an Article
        subject.should_not be_persisted
      end
    end
    ```

* It is acceptable to mock other models or child objects.
* Create the model for all examples in the spec to avoid duplication.

    ```Ruby
    describe Article do
      let(:article) { FactoryGirl.create(:article) }
    end
    ```

* Add an example ensuring that the FactoryGirl.created model is valid.

    ```Ruby
    describe Article do
      it 'is valid with valid attributes' do
        article.should be_valid
      end
    end
    ```

* When testing validations, use `have(x).errors_on` to specify the attibute
  which should be validated. Using `be_valid` does not guarantee that the 
  problem is in the intended attribute.

    ```Ruby
    # bad
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.should_not be_valid
      end
    end

    # preferred
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.should have(1).error_on(:title)
      end
    end
    ```

* Add a separate `describe` for each attribute which has validations.

    ```Ruby
    describe Article do
      describe '#title' do
        it 'is required' do
          article.title = nil
          article.should have(1).error_on(:title)
        end
      end
    end
    ```

* When testing uniqueness of a model attribute, name the other object 
  `another_object`.

    ```Ruby
    describe Article do
      describe '#title' do
        it 'is unique' do
          another_article = FactoryGirl.create(:article, title: article.title)
          article.should have(1).error_on(:title)
        end
      end
    end
    ```

### Mailers

* The model in the mailer spec should be mocked. The mailer should not depend on
  the model creation.
* The mailer spec should verify that:
  * the subject is correct
  * the receiver e-mail is correct
  * the e-mail is sent to the right e-mail address
  * the e-mail contains the required information

    ```Ruby
    describe SubscriberMailer do
      let(:subscriber) { mock_model(Subscription, email: 'johndoe@test.com', name: 'John Doe') }

      describe 'successful registration email' do
        subject { SubscriptionMailer.successful_registration_email(subscriber) }

        its(:subject) { should == 'Successful Registration!' }
        its(:from) { should == ['info@your_site.com'] }
        its(:to) { should == [subscriber.email] }

        it 'contains the subscriber name' do
          subject.body.encoded.should match(subscriber.name)
        end
      end
    end
    ```

# Contributing

Open tickets or send pull requests with improvements. Please write [good commit messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
or your pull requests will be closed.

# Credit
Inspiration was taken from the following:

[howaboutwe's rspec style guide](https://github.com/howaboutwe/rspec-style-guide)

[bbatsov's rspec style guide](https://github.com/bbatsov/rails-style-guide)