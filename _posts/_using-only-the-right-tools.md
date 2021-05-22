### Using only the "right" tools

There is a common phrase that sounds like this: "Use the right tool for the job". If you try to search for it on Quora, most people understand it the following way: if you want to drive a nail, use a hammer and not a screwdriver. And it makes sense, an inapropriate tool will take more time to do the work. It's hard to argue with such statement. However, the statement assumes that you know what is right and what is wrong. In software engineering it's usually not that clear.

Let's consider several examples to illustrate the complexity better: 

- Example 1. You develop an iOS app and thinking of defining the test strategy, specifically defining types of automated tests that you'd like to use (unit tests, UI tests, integration tests).
- Example 2. You have an old iOS app written completely in Objective-C. And thinking about rewriting it in Swift.

Can we surely say what's the right or wrong tool in the provided situations? I think it's difficult due to the lack of data to make a weighted decision. Each option has at the very least several important parameters, so we can think of the basic attributes like quality, price and speed.

Example #1. There would be a huge difference if you're a developing a bank or an audio player app. The expectations of correctness and the price of an error is very high when we're talking about money. So it makes more sense to put even extra automation testing for a banking app. While for an audio app, most of the users will tolerate minor problems if you deliver the core functionality and bring new useful features fast.

Example #2. To evaluate a migration to a new language, you'd need to know at the very least the expected lifetime of the app. Re-writing a project in Swift will take some time during which you'd not benefit from the new language, but receive quite some bugs and would need to fix them. So if an app doesn't have a future, it's just wasted effort and money. It helps to think about replacing an old technology with a new as a type of investment, where there is always a risk to lose when doing it without analysis.

In practice there are multiple factors that affect what is the best tool for the job. One tool that is right in one case, might be not in another. 

#### Why it's important?

There is a lot of speculation about what is the "best" or the "right" tool. Different companies, organizations and people are interested in promoting their products or services. The "right" tool is usually promoted as a superior and the only one, while all others are just worse. So while thinking about the "right" tool it makes sense to have your own definition of the best tool for your situation. So it's actually right tool for your job.

It takes some time and efforts. You'd need to learn what are the priorities and limitations for your project, find and compare the alternatives and choose the best without personal bias. As the result you'd get the decision that fits your situation the most. And also you'd learn more about the project and about decision making. 

Every software product an engineer builds is a sequence of choices that might be more or less optimal for the problem (UX problem, efficiency problem, etc). The more work you put into each decision the better would be the result. As a  skill it could be taught and learned, it's just a matter of practice.
