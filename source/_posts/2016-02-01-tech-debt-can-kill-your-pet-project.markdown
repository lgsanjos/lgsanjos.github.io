---
layout: post
title: "Tech debt demotivates and can kill your pet projects"
date: 2016-02-01 01:26:30 -0200
comments: true
draft: true
categories: tech
---

It's been over three years now that I work in a project called Meus Ativos (My Assets), this is a stock portifilio manager that helps long term investors to keep track of their stock's history and variation over the time.

First it began as a prototype, actually me and my brother were studying Android Development and were looking for simple ideas to code. It all started with a Android CRUD of transactions and did some math that most people do on Excel, it was then that we noticed the potential we had. After developing some code to keep the lastest price of each stock, we could calculate the gain/loss over time. We decided to keep moving the project forward but in a private repo, with some planning and guidelines, that's the reason we rebuild all the code this time using a ORM called **sugar ORM** and of course writting unit tests as it should be.

The app grew, we had over 5 screens and a few useful information we got a few downloads at** Google Play** altough the feedback was always positive, people asking for new features and enhancements.

After six months of development our productivity had dropped significantly, I felt like a snail, but what was wrong? We were just two developers, writing code with over 70% of test coverage and following a few conventions, how that was so unhealthy?

I remember taking some time to think about it, it's the luxury you have when there is no one paying to do the job, we  noticed a few problems with our code.

  1. The folder structure was the standard MVC: ./controllers/ ./models/ ./views/ ./services/ you know what I'm talking about
    That is a big problem in the long run, because you never think of this, "where is my controllers? Ahh of course they are in my controllers folder." we rarely look for the wires, we mostly look for business, "There is a bug on screen X. Question is: Where are the files related to that page?"

  2. Android's activity is a hard cookie, some times we had a few presentation rules that was not possible to be tested, and the real question is do we wanted to test it? With questions like that in mind, what was the correct approach?

  3. If the project continues to grow like this, what is going to happend to the project structure? Just create more folders and that's it? Can we create modules, can I compile just a piece of the system?

## Answers

Definatly the biggest problem was the architecture, aftere a while coding in Android we noticed a lot of effort on the presentation side and clearly our architecture was not handling that. So my brother came with a MVP (Model View Presentation) idea that would do it. It kept the presentation rules and the view itself totally separated and the models/services were completely independent.

Second step was that the folder structure should represent the App itself, so we created one folder for each feature, it didn't work very well because one screen could provide multiple features and that was the time to build modules and composing views. We didn't, the effort was quite high to manage multiple modules, our app was very small at that time it was unjustifiable.

The second other approach was to create a folder for each screen, think of the screen as a client or a delivery system, whatever this delivers is placed inside it's folder, sure we had multiple subfolders, so one for models again, other for activities and presenters and so on.

It was not a easy job, we had to hold the commits and refactore the architecture. One or two weeks later it was done and I'm proud of that work, it system kept growing without triggers concerns for us.

### Web version

We knew that storing this data on the person's phone wasn't a bright idea, people change phones and the app porpose (proposito) was to keep track of historic data, which means data from a long time ago whether should mean: Never lose data.

This came after one year of project, we had over 200 downloads on Google Play, I moved to a quite far city I had joined ThoughtWorks Brazil so I had less time to work on this, but I realised that I was learning more about Ruby On Rails there, and I wanted to keep practing it! There came the idea to build a website for:

  1. So we would have a landing page, to present our app.
  2. We could have a web version of the app, it would get the IPhone folks too.
  3. Cloud storage!

I remember that I start with no big plans and intentions, I just wanted to do a good kick start show it to my brother and see what we could achieve. So I started with a very simple landing page, based on a free bootstrap template. Added some screen shots, free pictures I found on internet it was not good whatsoever!

But he liked the idea and we kept moving on, so he was improving the landing page meanwhile I was build the app it self, I started with the database things, very simple things and the RoR productivity was insane, it droped only when working on UI, to get a beautifuld interface and a OK UX. That is hard!

Once we had a MVP of that thing, we decided to publish only the landing page, we wanted the users to use the web version synced with the mobile app, otherwise we would have two database completely out of syc. No sense at all.

It took me a month or more to build a sync algorithm, honestly it could have been done much faster than that, tough we had it in place wrote the sync code on the mobile, launched it first and a few days later launched the website. yay!

### What now?

Web development was so much better, and felt more rewarding as well, we had no motivation to add more features on mobile, first we won't deliver value to most of our users and the UI/UX on mobile is a concern if we add too much information is a pain to the user and for us to code.

The web became our thing, we kept adding features that we wanted for over 6 months, users were still asking for more features but we got tired. It's hard to keep motivated on side projects for too long, with no income.

I left ThoughtWorks one year later and I was not coding for over a few months, I noticed I was procrastinating too much, it was over the board, and the smarted move was to ask wwhy?

It also took a while for me to get an answer, but long story short, it was not fun anymore. System got big there were bad code here and there, lot's of complicated rules and the feeling was that any little thing would cost me a huge effort.

It certeanly rings a bell! This time I was working for **Skore.io** working on a Ruby on Rails team and previouly at TW I worked with AngularJS a lot, I got myself thinking that the UI would be much better as a single page application, it made totally sense to me. But rebuild again?

### Yes, rebuild!!!

We are always learning, we always can make things better, so lets do it better. One thing was bothering me, we were testing using minitest and mocha, I missed RSpec, it makes the test much more readable. In this mean time I learned a lot bout the active record and I knew that in the beginning we did a lot of SQL to go faster, I wanted to re do it all.

## Tests

When we first started we wanted to keep as simple as possible, and that's a sort of mistake. We skipped a good amount of gems that could saved us time, like factory girl, faker, RSpec itself and so one.

We had unit tests like this:

{% codeblock %}
class TradeTest < ActiveSupport::TestCase

  test "Create new" do
      t = Trade.new
      t.stock_name = "PETR3"
      t.price = 15.30
      t.amount = 200
      t.user_id = 2
  end

  test 'stock name is always upcased' do
      t = Trade.new
      t.stock_name = "hgtx3"
      assert_equal 'HGTX3', t.stock_name
  end

  test 'trade should be stock' do
	t = Trade.new
      t.stock_name = "PETR3"
      assert t.stock?
      refute t.property_fund?
  end

  test 'trade should be property found' do
	t = Trade.new
      t.stock_name = "XPGA11"
      refute t.stock?
      assert t.property_fund?
  end

  ...

  test 'current_price should invoke stock_price' do
      quote = OpenStruct.new
      quote.last_trade_price = 16.80

      MyAssetsFinanceCache.expects(:quote).with('PETR3').once.returns(quote)

      t  = Trade.new
      t.stock_name = 'PETR3'
      t.price = 13.90

      assert_equal 16.80, t.current_price
  end
end 
{% endcodeblock %}

See, big tests, always creating declaratively the models and even mocking up it's relationships.
It's better to leave all this buiding thing to factory girl, which by the way is much faster, not always you want to save data.

{% codeblock %}
describe Trade do
  subject { build(:trade) }

  # Relationship
  it { is_expected.to have_many(:action) }
  it { is_expected.to belong_to(:user) }

  # Validation
  it { is_expected.to validate_presence_of(:stock_name) }
  it { is_expected.to validate_presence_of(:trade_type) }
  it { is_expected.to validate_presence_of(:date) }
  it { is_expected.to validate_presence_of(:user) }
  it { is_expected.to validate_presence_of(:amount) }
  it { is_expected.to validate_presence_of(:price) }

  # Scope

  describe '#of_user' do
    it 'includes all trades with same user_id' do
       user = build(:user, id: 2)
       trade = create(:trade, id: 2, user: user)
       expect(Trade.of_user(2).size).to be_eql 1
       expect(Trade.of_user(2)).to include trade
    end

    it 'excludes all trades with different user_id' do
       user = build(:user, id: 1)
       trade = create(:trade, id: 2, user: user)
       expect(Trade.of_user(3)).to be_empty
    end
  end

  # Methods

  describe '#stock_name' do
    it 'is always upcased' do
      subject.stock_name = 'PeTr3'
      expect(subject.stock_name).to be_eql 'PETR3'
    end
  end

  describe '#stock?' do
    it 'is true when stock_name dont contains a 11' do
        expect(subject.stock?).to be_truthy
    end

    it 'is false when stock_name contains a 11' do
        subject.stock_name = 'BPFF11'
        expect(subject.stock?).to be_falsy
    end
  end

  describe '#property_fund?' do
    subject { build(:trade, stock_name: 'SAAG11') }

    it 'is true when stock_name contains a 11' do
        expect(subject.property_fund?).to be_truthy
    end

    it 'is false when stock_name does not contains a 11' do
        expect(subject.property_fund?).to be_falsy
    end

    it 'is true when stock_name contains a 11B' do
        subject.stock_name = 'BPFF11B'
        expect(subject.property_fund?).to be_truthy
    end
  end

end
{% endcodeblock %}

RSpec is very readable on long term, so it's clear the class tests, each method tests, the scope if very clear and it will make your life less miserable later on.

### Models

Models are now following a sequence like:

  . attr_accessor
  . validations
  . callbacks
  . scope
  . class methods
  . public methods
  . private

All the business rules were extracted to auxiliary classes, so if I need to calculate the average price of a particular stock, there is a class that will receives a stock and the inital date. It allows the model to breath, I know all that discussion about anemic model vs fat models, I believe thin models are much more sustainable in long term. If every new feature you keep adding methods in the mode class, it won't stand a chance.

### Controllers

Now the controllers are simple APIs they just receive and responde JSON content, no more html things. It turns the life much easier it hard to explain but it does for now.

### UI

In fact we are rebuilding the UI to an Angular project, this way we get the single page application just consuming the service and that's what I call by placing a big wall between business and presentation.

## Upcoming challanges

  . The more you split into pieces hards is to manage, build, deploy everything. This will be my concern soon!
  . We will need some integration tests, probably with capybara, it should come out of the box soon.
  . Internationalization wasn't a big hit, but now it must be.
