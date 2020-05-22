+++
title = "Development Flow"
date = 2020-05-08T13:06:22+01:00
weight = 2
draft = false
+++

## DevOps (addendum)

In the olden days Facebook had a really cool engineering slogan __Move fast, break things__. DevOps goes in a different
direction. They dropped the __break things__ stigma, not because we slowed down but we started breaking things in
controlled environments. We try out our changes in production scale replicas.

As a rule of thumb your SDLC should look something like this
{{<mermaid>}}
graph LR
    id1("Commit stage (automated)") ==> id2("Acceptance stage (automated)")
    id1("Commit stage (automated)") ==> id4("Exploratory testing (manual)")
    id2("Acceptance stage (automated)") ==> id5("Production (manual)")
    id2("Acceptance stage (automated)") ==> id3("Capacity testing (automated)")
    id2("Acceptance stage (automated)") ==> id6("UAT (manual)")
    linkStyle 0 stroke-width:5px,fill:none,stroke:blue;
    linkStyle 3 stroke-width:5px,fill:none,stroke:blue;
{{</mermaid>}}

The lines for testing are a little blurred when it comes to front end applications.

One way to look at it would be:
- Commit stage: component library - unittests + snapshot tests
- Acceptance: succesfully integrated into app and visual tests are run
- the rest are pretty straightforward

## Multi layered testing
#### Unittests
Tests for functions and classes, in isolation, they don't require the API or any other dependencies
#### Acceptance
Tests that are of higher level functionality, things that describe user stories and required features
#### Integration
Tests that check that our feature has been included in our app and has not caused unintended side effects

{{% notice note %}}
Until now we have had a look at unit and acceptance tests. We tested sagas through acceptance tests, selectors and
reducers with unittests. In the next section we will have a look at
[cypress](https://docs.cypress.io/guides/tooling/visual-testing.html#Functional-vs-visual-testing)
{{% /notice %}}

