# ReachLocal: RSpec Style Guide

> **Learn the rules like a pro, so you can break them like an artist.** - _Pablo Picasso_

This guide helps describe some recommended practices that we've found
to help in writing specs that are easier for ourselves and our colleagues
to comprehend and maintain.

Although, maybe you've found something that doesn't make sense in your
current context? Maybe there's something missing in the guideline? Progress
is more important than doctrine so move forward then loop back. There might
be a reason things were recommended here or it might be something that was missed.

If you have an idea on how to improve this doc then please submit a PR with
an explanation of what you're changing with an explanation of your goal.

## Recommendations

### Line Returns after `feature`, `context`, or `describe`

Avoid line returns after `feature`, `context` or `describe`
descriptions. It does makes the code more difficult to read and lowers the
value of logical chunks.

#### Not recommended

```ruby
describe Article do

  describe '#summary' do

    context 'when there is a summary' do

      it 'returns the summary' do
        # ...
      end
    end
  end
end
```

#### Recommended

```ruby
describe Article do
  describe '#summary' do
    context 'when there is a summary' do
      it 'returns the summary' do
        # ...
      end
    end
  end
end
```

### `let`, `subject`, and `before`/`after` group line returns

Leave one line return after `let`, `subject`, and `before`/`after` blocks.

#### Not recommended

```ruby
describe Article do
  subject { FactoryGirl.create(:some_article) }
  describe '#summary' do
    # ...
  end
end
```

#### Recommended

```ruby
describe Article do
  subject { FactoryGirl.create(:some_article) }

  describe '#summary' do
    # ...
  end
end
```

### `let`, `subject`, `before`/`after` grouping

Only group `let`, `subject` blocks and separate them from `before`/`after`
blocks. It makes the code much more readable.

#### Not recommended

```ruby
describe Article do
  subject { FactoryGirl.create(:some_article) }
  let(:user) { FactoryGirl.create(:user) }
  before do
    # ...
  end
  after do
    # ...
  end
  describe '#summary' do
    # ...
  end
end
```

#### Recommended

```ruby
describe Article do
  subject { FactoryGirl.create(:some_article) }
  let(:user) { FactoryGirl.create(:user) }

  before do
    # ...
  end

  after do
    # ...
  end

  describe '#summary' do
    # ...
  end
end
```

### `it` block line returns

Leave one line return around `it` blocks. This helps to separate the
expectations from their conditional logic (contexts for instance).

#### Not recommended

```ruby
describe '#summary' do
  let(:item) { double('something') }

  it 'returns the summary' do
    # ...
  end
  it 'does something else' do
    # ...
  end
  it 'does another thing' do
    # ...
  end
end
```

#### Recommended

```ruby
describe '#summary' do
  let(:item) { double('something') }

  it 'returns the summary' do
    # ...
  end

  it 'does something else' do
    # ...
  end

  it 'does another thing' do
    # ...
  end
end
```

### `before(:each)`or `before`

There is no need to specify `(:each)` for `before`/`after` blocks, as it is
the default functionality. There are almost zero cases to use `before(:all)`
anymore, but if you do find one - just write it out like `before(:all)`

#### Not recommended

```ruby
describe '#summary' do
  before(:each) do
    subject.summary = 'something'
  end
end
```

#### Recommended

```ruby
describe '#summary' do
  before do
    subject.summary = 'something'
  end
end
```

### 'should' it or 'should not' in `it` statements

Avoid writing 'should' or 'should not' in the beginning of your `it` blocks.
The descriptions represent actual functionality - not what might be happening.

#### Not recommended

```ruby
it 'should return the summary' do
  # ...
end
```

#### Recommended

```ruby
it 'returns the summary' do
  # ...
end
```

### The One Expectation

Use only one expectation per example. There are very few scenarios where two
or more expectations in a single `it` block should be used. So, general rule
of thumb is one expectation per `it` block.

#### Bad Example

```ruby
describe ArticlesController do
  #...

  describe 'GET new' do
    it 'assigns new article and renders the new article template' do
      get :new
      expect(assigns[:article]).to be_a(Article)
      expect(response).to render_template :new
    end
  end

  # ...
end
```

#### Good Example

```ruby
describe ArticlesController do
  #...

  describe 'GET new' do
    it 'assigns a new article' do
      get :new
      expect(assigns[:article]).to be_a(Article)
    end

    it 'renders the new article template' do
      get :new
      expect(response).to render_template :new
    end
  end
end
```

### Context Cases

`context` blocks should pretty much always have an opposite negative case. It
should actually be a strong code smell if there is a single context (without a
matching negative case) that it needs refactoring, or may have no purpose.

#### Not recommended

```ruby
# This is a case where refactoring is the correct choice
describe '#attributes' do
  context 'the returned hash' do
    it 'includes the display name' do
      # ...
    end

    it 'includes the creation time' do
      # ...
    end
  end
end

# This is a case where the negative case needs to be tested, but wasn't
describe '#attributes' do
  context 'when display name is present' do
    before do
      subject.display_name = 'something'
    end

    it 'includes the display name' do
      # ...
    end
  end
end
```

#### Recommended

```ruby
# Refactored
describe '#attributes' do
  subject { FactoryGirl.create(:article) }

  expect(subject.attributes).to include subject.display_name
  expect(subject.attributes).to include subject.created_at
end

# Added the negative case
describe '#attributes' do
  context 'when display name is present' do
    before do
      subject.display_name = 'something'
    end

    it 'includes the display name' do
      # ...
    end
  end

  context 'when display name is not present' do
    before do
      subject.display_name = nil
    end

    it 'does not include the display name' do
      # ...
    end
  end
end
```

### `context` descriptions

`context` block descriptions should always start with 'when', and be in the
form of a sentence with proper grammar.

#### Not recommended

```ruby
context 'the display name not present' do
  # ...
end
```

#### Recommended

```ruby
context 'when the display name is not present' do
  # ...
end
```

### `it` descriptions

`it` block descriptions should never end with a conditional. This is a code
smell that the `it` most likely needs to be wrapped in a `context`.

#### Not recommended

```ruby
it 'returns the display name if it is present' do
  # ...
end
```

#### Recommended

```ruby
context 'when display name is present' do
  it 'returns the display name' do
    # ...
  end
end

# This encourages the addition of negative test cases that might have
# been overlooked
context 'when display name is not present' do
  it 'returns nil' do
    # ...
  end
end
```

### `describe` block naming

- use hash '#method' for instance methods
- use dot '.method' for class methods

Given the following exists

```ruby
class Article
  def summary
    # ...
  end

  def self.latest
    # ...
  end
end
```

#### Not recommended

```ruby
describe Article do
  describe 'summary' do
    #...
  end

  describe 'latest' do
    #...
  end
end
```

#### Recommended

```ruby
describe Article do
  describe '#summary' do
    #...
  end

  describe '.latest' do
    #...
  end
end
```

### `it` in iterators

Avoid writing iterators to generate tests. When another developer adds a
feature to one of the items in the iteration, he must then break it out into a
separate test - he is forced to edit code that has nothing to do with his pull
request.

#### Not recommended

```ruby
[:new, :show, :index].each do |action|
  it 'returns 200' do
    get action
    expect(response).to be_ok
  end
end
```

#### Recommended

more verbose for the time being, but better for the future development

```ruby
describe 'GET new' do
  it 'returns 200' do
    get :new
    expect(response).to be_ok
  end
end

describe 'GET show' do
  it 'returns 200' do
    get :show
    expect(response).to be_ok
  end
end

describe 'GET index' do
  it 'returns 200' do
    get :index
    expect(response).to be_ok
  end
end
```

### Factories/Fixtures

Use [Factory Girl](https://github.com/thoughtbot/factory_girl) to create test
objects in integration tests. You should very rarely have to use
`ModelName.create` within an integration spec. Do **not** use fixtures as they
are not nearly as maintainable as factories.

```ruby
subject { FactoryGirl.create(:some_article) }
```

### Mocks/Stubs/Doubles

Use mocks and stubs with caution. While they help to improve the performance
of the test suite, you can mock/stub yourself into a false-positive state very
easily. When resorting to mocking and stubbing, only mock against a small,
stable, obvious (or documented) API, so stubs are likely to represent reality
after future refactoring.

This generally means you should use them with more isolated/behavioral
tests rather than with integration tests.

```ruby
# double an object
article = double('article')

# stubbing a method
allow(Article).to receive(:find).with(5).and_return(article)
```

*NOTE*: if you stub a method that could give a false-positive test result, you
have gone too far. See below:

#### Not recommended

```ruby
subject { double('article') }

describe '#summary' do
  context 'when summary is not present' do
    # This stubbing of the #nil? method, makes the test pass, but
    # you are no longer testing the functionality of the code,
    # you are testing the functionality of the test suite.
    # This test would pass if there was not a single line of code
    # written for the Article class.
    it 'returns nil' do
      summary = double('summary')
      allow(subject).to receive(:summary).and_return(summary)
      allow(summary).to receive(:nil?).and_return(true)
      expect(subject.summary).to be_nil
    end
  end
end
```

#### Recommended

```ruby
subject { double('article') }

describe '#summary' do
  context 'when summary is not present' do
    # This is no longer stubbing all of the functionality, and will
    # actually test the objects handling of the methods return value.
    it 'returns nil' do
      allow(subject).to receive(:summary).and_return(nil)
      expect(subject.summary).to be_nil
    end
  end
end
```

### Dealing with Time

Always use [Timecop](https://github.com/travisjeffery/timecop) instead of
stubbing anything on Time or Date.

#### Not recommended

```ruby
it 'offsets the time 2 days into the future' do
  current_time = Time.now
  allow(Time).to receive(:now).and_return(current_time)
  expect(subject.get_offset_time).to be_the_same_time_as (current_time + 2.days)
end
```

#### Recommended

```ruby
it 'offsets the time 2 days into the future' do
  Timecop.freeze(Time.now) do
    expect(subject.get_offset_time).to be_the_same_time_as 2.days.from_now
  end
end
```

*NOTE*: `#be_the_same_time_as` is a RSpec matcher we added to the platform, it
is not normally available to RSpec.

### `let` blocks

Use `let` blocks instead of `before(:each)` blocks to create data for the spec
examples. `let` blocks get lazily evaluated. It also removes the instance
variables from the test suite (which don't look as nice as local variables).

These should primarily be used when you have duplication among a number of
`it` blocks within a `context` but not all of them. Be careful with overuse of
`let` as it makes the test suite much more difficult to read.

```ruby
# use this:
let(:article) { FactoryGirl.create(:article) }

# ... instead of this:
before { @article = FactoryGirl.create(:article) }
```

### `subject`

Use `subject` when possible

```ruby
describe Article do
  subject { FactoryGirl.create(:article) }

  it 'is not published on creation' do
    expect(subject).not_to be_published
  end
end
```

### Magic Matchers

Use RSpec's 'magical matcher' methods when possible. For instance, a class
with the method `published?` should be tested with the following:

```ruby
it 'is published' do
  # actually tests subject.published? == true
  expect(subject).to be_published
end
```

### Incidental State

Avoid incidental state as much as possible.

#### Not recommended

```ruby
it 'publishes the article' do
  article.publish

  # Creating another shared Article test object above would cause this
  # test to break
  expect(Article.count).to eq(2)
end
```

#### Recommended

```ruby
it 'publishes the article' do
  expect { article.publish }.to change(Article, :count).by(1)
end
```

### DRY

Be careful not to focus on being 'DRY' by moving repeated expectations into a
shared environment too early, as this can lead to brittle tests that rely too
much on one other.

It general it is best to start with doing everything directly in your `it`
blocks even if it is duplication and then refactor your tests after you have
them working to be a little more DRY. However, keep in mind that duplication
in test suites is NOT frowned upon, in fact it is preferred if it provides
easier understanding and reading of a test.

### Views

* The directory structure of the view specs `spec/views` matches the
  one in `app/views`. For example the specs for the views in
  `app/views/users` are placed in `spec/views/users`.
* The naming convention for the view specs is adding `_spec.rb` to the
  view name, for example the view `_form.html.erb` has a
  corresponding spec `_form.html.erb_spec.rb`.
* `spec_helper.rb` needs to be required in each view spec file.
* The outer `describe` block uses the path to the view without the
  `app/views` part. This is used by the `render` method when it is
  called without arguments.

    ```ruby
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

    ```ruby
    # spec/views/articles/edit.html.erb_spec.rb
    describe 'articles/edit.html.erb' do
      it 'renders the form for a new article creation' do
        assign(:article, double(Article).as_null_object)
        render
        expect(rendered).to have_selector('form',
          method: 'post',
          action: articles_path
        ) do |form|
          expect(form).to have_selector('input', type: 'submit')
        end
      end
    end
    ```

* Prefer the capybara negative selectors over to_not with the positive.

    ```ruby
    # bad
    expect(page).to_not have_selector('input', type: 'submit')
    expect(page).to_not have_xpath('tr')

    # good
    expect(page).to have_no_selector('input', type: 'submit')
    expect(page).to have_no_xpath('tr')
    ```

* When a view uses helper methods, these methods need to be stubbed. Stubbing
  the helper methods is done on the `template` object:

    ```ruby
    # app/helpers/articles_helper.rb
    class ArticlesHelper
      def formatted_date(date)
        # ...
      end
    end
    ```

    ```Rails
    # app/views/articles/show.html.erb
    <%= 'Published at: #{formatted_date(@article.published_at)}' %>
    ```

    ```ruby
    # spec/views/articles/show.html.erb_spec.rb
    describe 'articles/show.html.erb' do
      it 'displays the formatted date of article publishing' do
        article = double(Article, published_at: Date.new(2012, 01, 01))
        assign(:article, article)

        allow(template).to_receive(:formatted_date).with(article.published_at).and_return('01.01.2012')

        render
        expect(rendered).to have_content('Published at: 01.01.2012')
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

    ```ruby
    # Example of a commonly used controller spec
    # spec/controllers/articles_controller_spec.rb
    # We are interested only in the actions the controller should perform
    # So we are mocking the model creation and stubbing its methods
    # And we concentrate only on the things the controller should do

    describe ArticlesController do
      # The model will be used in the specs for all methods of the controller
      let(:article) { double(Article) }

      describe 'POST create' do
        before { allow(Article).to receive(:new).and_return(article) }

        it 'creates a new article with the given attributes' do
          expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
          post :create, message: { title: 'The New Article Title' }
        end

        it 'saves the article' do
          expect(article).to receive(:save)
          post :create
        end

        it 'redirects to the Articles index' do
          allow(article).to receive(:save)
          post :create
          expect(response).to redirect_to(action: 'index')
        end
      end
    end
    ```


* Use context when the controller action has different behaviour depending on
  the received params.

    ```ruby
    # A classic example for use of contexts in a controller spec is creation or update when the object saves successfully or not.

    describe ArticlesController do
      let(:article) { double(Article) }

      describe 'POST create' do
        before { allow(Article).to receive(:new).and_return(article) }

        it 'creates a new article with the given attributes' do
          expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
          post :create, article: { title: 'The New Article Title' }
        end

        it 'saves the article' do
          expect(article).to receive(:save)
          post :create
        end

        context 'when the article saves successfully' do
          before do
            allow(article).to receive(:save).and_return(true)
          end

          it 'sets a flash[:notice] message' do
            post :create
            expect(flash[:notice]).to eq('The article was saved successfully.')
          end

          it 'redirects to the Articles index' do
            post :create
            expect(response).to redirect_to(action: 'index')
          end
        end

        context 'when the article fails to save' do
          before do
            allow(article).to receive(:save).and_return(false)
          end

          it 'assigns @article' do
            post :create
            expect(assigns[:article]).to eq(article)
          end

          it 're-renders the 'new' template' do
            post :create
            expect(response).to render_template('new')
          end
        end
      end
    end
    ```

### Models

* Don't mock the models in their own specs.
* Use `FactoryGirl.create` to make real objects, or just use a new (unsaved)
  instance with `subject`.

    ```ruby
    describe Article do
      let(:article) { FactoryGirl.create(:article) }

      # Currently, 'subject' is the same as 'Article.new'
      it 'is an instance of Article' do
        expect(subject).to be_an Article
      end

      it 'is not persisted' do
        expect(subject).to_not be_persisted
      end
    end
    ```

* It is okay to mock other models or child objects.
* Create the model for all examples in the spec to avoid duplication.

    ```ruby
    describe Article do
      let(:article) { FactoryGirl.create(:article) }
    end
    ```

* Add an example ensuring that the FactoryGirl.created model is valid.

    ```ruby
    describe Article do
      it 'is valid with valid attributes' do
        expect(article).to be_valid
      end
    end
    ```


* When testing validations, use `expect(model.errors[:attribute].size).to eq(x)` to specify the attribute
  which should be validated. Using `be_valid` does not guarantee that the
  problem is in the intended attribute.

    ```ruby
    # bad
    describe '#title' do
      it 'is required' do
        article.title = nil
        expect(article).to_not be_valid
      end
    end

    # preferred
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.valid?
        expect(article.errors[:title].size).to eq(1)
      end
    end
    ```

* Add a separate `describe` for each attribute which has validations.

    ```ruby
    describe Article do
      describe '#title' do
        it 'is required' do
          article.title = nil
          article.valid?
          expect(article.errors[:title].size).to eq(1)
        end
      end
    end
    ```

* When testing uniqueness of a model attribute, name the other object
  `another_object`.

    ```ruby
    describe Article do
      describe '#title' do
        it 'is unique' do
          another_article = FactoryGirl.create(:article, title: article.title)
          article.valid?
          expect(another_article.errors[:title].size).to eq(1)
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

    ```ruby
    describe SubscriberMailer do
      let(:subscriber) { double(Subscription, email: 'johndoe@test.com', name: 'John Doe') }

      describe 'successful registration email' do
        subject { SubscriptionMailer.successful_registration_email(subscriber) }

        its(:subject) { should == 'Successful Registration!' }
        its(:from) { should == ['info@your_site.com'] }
        its(:to) { should == [subscriber.email] }

        it 'contains the subscriber name' do
          expect(subject.body.encoded).to match(subscriber.name)
        end
      end
    end
    ```

# Contributing

Questions about this doc? Have something that you'd like to see changed but
aren't ready for a PR? Then submit an issue to this repo to start a conversation.

Want to dive right in and start changing the doc? Then PR's are welcome. If
you're unsure about how to format a PR then we recommend reading about how to
write [good commit messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

# Credit

Inspiration was taken from the following:

- [howaboutwe's rspec style guide](https://github.com/howaboutwe/rspec-style-guide)
- [bbatsov's rspec style guide](https://github.com/bbatsov/rails-style-guide)
