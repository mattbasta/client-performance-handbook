# Prioritizing Performance Improvements

Now that you know what to look at and why you should be looking, let's talk about what to do when you've found something interesting.

It's easy to want to fix every problem that comes along, or to want to skip over hard problems and instead take care of some low-hanging fruit. How should performance issues be addressed? There are a number of factors to consider:

- **How many users are affected by the issue?** If the benefits of the change are limited, it may be more valuable to push it off and work on something more impactful. On the other hand, if the benefit is substantial and the change is easy to make, it makes sense to go ahead with it.
- **How much benefit is there?** A change that impacts a very large number of users but doesn't provide much benefit may not be worth as much as a change that provides more benefit to a smaller number of users. Targeted fixes can provide much needed improvements that fix substantial user frustrations.
- **Which users are affected?** Sometimes it's hard to determine where attention should be focused. On one hand, it's important to benefit the largest number of users with the biggest wins possible. However, it may be more fiscally responsible to make changes that benefit paid users before focusing on fixes that impact free users.
- **How much does the change cost?** Not all performance improvements are free. Adding a CDN, for instance, may significantly increase hosting costs. If the costs involved outweigh the benefits that the improvement will provide, it may be more worthwhile to look elsewhere.
- **What other improvements does the change unlock?** If making the change (regardless of how small it may be) unlocks the ability to make other worthwhile performance improvements, it may be a valuable change to make.

In general, you can model how worthwhile a change is using the following formula:

$$w = \frac{N W}{D C}$$

- **N:** The number of users that will see an improvement
- **W:** The expected wins
- **D:** The difficulty, or amount of time involved in making the change
- **C:** The (fiscal) cost of the change

Obviously, the units of each variable are subjective. Measuring difficulty, for example, might be better done in man hours for one team and sprints for another. Expected wins might be measured in dollars, engagement, or milliseconds. The dimensional analysis required to make this approach useful may not be simple and will vary greatly between organizations. A good product manager should use a similar approach to determine what areas of performance deserve the most attention.