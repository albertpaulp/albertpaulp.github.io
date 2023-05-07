---
layout: post
title:  Building a reusable questionnaire framework
date:   2022-04-08 14:07:23 +0530
categories: software-design
---

*This is a [cross post](https://clark.engineering/asking-the-right-questions-44c4e8188849) from Clark engineering blog written by me.*

At [Clark](https://www.clark.de/), our mission is to deliver peace of mind to our customers. While universally desirable, this is a highly subjective matter. To recommend the right insurance cover, it is critically important to understand the customer’s life situation. We thus need to ask our customers the right questions and collect reliable data to provide the best insurance tailored to our customer’s needs.

*This is a story about how we created a framework called "Census" to ask the right questions and collect the data.*

Initially, when I heard about this requirement, I was thinking: how hard could it be to create a template of questions, ask and collect answers? While this could be solved by a simple static questionnaire, this wasn’t remotely close to what we needed. After interviewing business stakeholders and going through customer research, we had more concrete requirements. Here are some of the main requirements we collected:

- **No-code solution**: Business analysts would like to design/deploy/maintain questionnaires to customers without developer effort.
- **Dynamic skip logic**: The framework should be "intelligent" enough to skip questions based on the customer’s previous answers, eg. not asking about the customer’s car details if the customer doesn’t own one.
- **Answer validations**: Ability to validate answers given by customers so that we can ensure collected data is of the highest quality. Eg. age must be an integer
- **Reasonably general purpose**: The framework should be reusable for different kinds of questionnaires. Eg. for a very specific category of insurance, NPS, Surveys etc.
- **Play/Pause**: Customers should be able to pause and resume answers across sessions/devices.
- **GDPR/Security**: Collected data should be kept in-house database.
- **Measuring/Tracking**: Measuring and tracking the progress of customers, understanding which questions are not well received by customers etc.
- **Framework as a library**: The framework should be usable across Clark applications, ideally a library that can be added with minimal integration effort.

### Enter the ‘make or buy’ decision
Now that we had a list of solid requirements, we entered the make or buy phase — and were determined to find a SaaS tool that fits our needs. There are plenty of Questionnaire SaaS tools in the market but none fit our criteria, notably on things like “on-premise data” and custom business-oriented validations for answers. So we decided to build one, and aimed at a proof of concept before investing more resources into this project. The project was named ‘**Census**’ after the ancient practice of data collection in Rome.

### The first challenge: Defining structure
The first challenge was to find a suitable data structure to express the Questionnaire. After a little bit of brainstorming, we could see we needed a non-linear data structure like the *tree* to represent questions in a questionnaire. Even though the linked list could be a candidate, we ruled it out since it’s possible to have cycles in the linked list (or we have to do cycle detection). General Tree data structure lets us easily traverse through questions, branch out, skip questions etc. The *tree* data structure is also guaranteed not to have any cycles by design, we wouldn’t want our customers stuck in a loop answering questions for eternity.

![questionnaire tree structure](/assets/images/building-questionnaire-framework/census-1.webp)

*A simple questionnaire is expressed in the tree structure.*

### Building a no-code / low-code solution
The next challenge was to create and maintain questionnaires dynamically. One of our requirements was a no-code/low-code solution so that our business partners can deploy/maintain questionnaires without engineering support. We found out some of our business partners are already comfortable reading/editing YAML files. We thus realized that we could skip building a UI for creating templates in the beginning and use YAML files to express templates. Business partners would write their questionnaires in YAML files and they can easily upload them to create Questionnaires without any developer effort! So we created our own DSL (Domain Specific Language) for creating templates.

```yml
# sample Questionnaire Template in yaml for questionnaire in above.
template:
  name: referral_rewards
  
  fields:
    - field: &customer_name 
        name: customer_name
        datapoint: customer_name
        input_type: text
        value_type: string
        label: What is your name ?
        description: We need your name to ship some rewards !
        validations:
          min_length: 1
          max_length: 25
    .............(Skipping rest of fields)          
  flows:
    default:
      - field: *customer_name
        name: customer_name
        required: true
        next:
         - field: *customer_age
           name: age
           required: true
           next:
             - field: *beer_type
               name: beer_type
               required: false
               conditions: "customer_age >= 18"
             - field: *candy_type
               name: candy_type
               required: false
               conditions: "customer_age < 18"
```

Let me explain some new terminologies in this DSL.

To start with, you can think of template as a blueprint of a questionnaire. form is an instance of a customer attempting to answer a template. flows are used to have multiple questionnaires flow per template. Consider running an A/B experiment with the same template, we might need different orders of asking questions to understand which ones are the best. datapoint is a mechanism to generalize collected data so that it can be reused outside of a template. validations are validations! required attribute in flows shows if that question can be skipped by the customer. conditions are expressions that can be evaluated, the customer will be allowed to answer that node if the condition expression is evaluated as true.

We can think of template creation in 3 components.

![census-components](/assets/images/building-questionnaire-framework/census-2.webp)

#### Converting the questionnaire converter into a Ruby Gem
What’s left is mostly coding a ‘*YAML-to-tree-converter*’ and a component to store questionnaire tree structure to the database. We created a Rails engine that can be mounted as a gem in Ruby (since our main application is powered by Rails). Now we can use this gem with any application where we need to run questionnaires to customers. Business partners use ‘Census’ via an admin panel that can ingest template files and create templates in the database. For brevity, I am omitting a lot of nuances (auth, storing tree structure in DB) of the building.

#### Customer-facing APIs for Questionnaires
‘Census’ gem provides 3 APIs for handling questionnaires to hosted applications. The first one is to create a form for customers, the second one is to collect answers and the last one is to finish the form.

### Key takeaways
Census is now powering most of the questionnaires in production and we are adding more questionnaires with it every day. These are the few key takeaways from this project:

- Picking the right data structures simplifies downstream work.
- Building proof of concept(prototyping) before actual implementation can help us de-risk time and effort.
- Building frameworks as reusable components can save a lot of time when we need to implement them in other places.

Feel free to ask any *questions(No pun intended)*!