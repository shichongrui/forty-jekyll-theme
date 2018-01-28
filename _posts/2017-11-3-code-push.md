---
title: CodePush, is it worth it?
description: What is CodePush and is it worth integrating into your React Native App
image: /assets/images/codepush.jpg
layout: post
---

Last week at work was hackweek, so I took the time to look into getting CodePush into our React Native app. It was pretty simple to get setup but after some discussion with the team we started to question whether or not CodePush was worth the added complexity.

So what is CodePush? CodePush is a service provided by Microsoft for Cordova and React Native apps, that allows you to push out new versions of your javascript bundle to your app users, **without** having to go through an Apple review. This immediately sounds appealing considering that if you have a critical bug fix you don't have to wait for Apple to review the app before it can go out to users.  Implementing it into our app was pretty straight forward and instructions for doing so can be found on their website [https://microsoft.github.io/code-push/](https://microsoft.github.io/code-push/).

CodePush can be super useful for small apps and developer teams who may not have as much complexity as the app I tried to put CodePush into. I would encourage anyone to try it out and see if it works for you. After implementing CodePush and pushing up a pull request for our app we had a team discussion about what CodePush would mean and we determined that it might add some unnecessary complexity as well as possibly not even being needed.

### Schemes and Builds
The first of such complexities was the necessity for altering our automated builds and app schemes. In a React Native app you typically will have 2 schemes, one for development, and the other for releasing to the app store, though I use both schemes for different purposes during development. With CodePush you have to setup the native side using the CodePush SDK to grab new js bundles from their service. The app will reach out to the CodePush service on app launch and check if there are any js bundles that are intended for the current app version, download the bundle, and then use it the next time the user opens the app. This means that we need the scheme that we release to the App Store to include the CodePush workflow. But if we do that to our existing release scheme that means I won't be able to check out my local changes work because every time I open the app, CodePush will try to bring down whatever is on CodePush. So then I need to create a new scheme that uses CodePush, create a new build in our automated build platform, and have that one be the one we release to iTunes Connect. A bit of a pain.

We don't stop there though. CodePush has 2 different environments by default; Production and Staging. So we need to create another new scheme from the original release scheme and set it up to point to CodePush's Staging environment so that our QA engineers can ensure that what we deploy to CodePush works the way we intended it to. So we had to double the number of Schemes we have and change our builds in our automated build platform to get this working. Not ideal.

### Git workflow
Our automated builds can be triggered conditionally based on changes in the git pull request as well as based on branch naming schemes. After we have created the necessary builds to handle CodePush we have to setup conditional triggering based on the pull request that was submitted. In order to get this right it requires adhering to a strict git workflow for bug fixes. This is a net gain overall since regardless of how you release code to users, there should be some git workflow in place to ensure that only bug fixes go out in patches to users. Don't want to prematurely ship unfinished feature work to users. So that's fine, we may be doing it one way now and CodePush might require doing it a new way. But the real problem arises for QA engineers when they have to be aware of what kind of release it is. If it's a release that needs to go to the app store it's going to be a different process than one that is going out to CodePush Staging environment and then ultimately the Production environment. It may not seem like a big deal for QA to have to be aware of this, but the whole point of automated builds is to remove the need for human intervention and the need for humans to have to remember process and procedures. If you can keep the amount of complexity to just having to remember to do things one way, the chances for bad things to happen reduce.

### When do users see the hot fix
This is possibly the last nail in the coffin for us. CodePush is intended to help you get around Apple review times and immediately get bug fixes out to users. But is it immediate? As a team we discussed the ideal times we wanted CodePush to apply the new js bundle and how long it would take for a user to see that change. CodePush gives you a lot of flexibility of deciding when it reaches out to it's service to request new js bundles and when to apply them. We decided that the only safe time to do that was when the app launches from being closed. If you do it any more often than that such as immediately or even when the app returns from being backgrounded, you risk interrupting users in what they are doing at the time. I might switch apps to respond to a text message, then immediately switch back to my app, in which case CodePush may have applied a new js bundle and essentially restarted the app, causing me to loose whatever it was I was working on.  So then how often do users end up having the app start up from scratch? Probably once a day if not less frequent than that. So even if we pushed a change out to CodePush it would take more than a day for the user to get the change? Ok well that must still be better than Apple review times right? Not anymore. [1 day on average](http://appreviewtimes.com/). Now yes this data isn't official but it's a community effort to track Apple review times. We had a critical native code bug fix that we pushed to iTunes Connect and requested expedited review on. 3 hours later we released the bug fix to the app store.

CodePush may make sense for some teams and apps. For us it was more complex than the benefits our users would actually get from it. Where Apple review times are continually improving, the main benefit of CodePush becomes unneccessary.