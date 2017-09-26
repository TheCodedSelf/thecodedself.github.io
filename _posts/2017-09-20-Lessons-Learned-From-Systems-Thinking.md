---
title: "Lessons learned from Systems Thinking"
permalink: /systems-thinking/
---

I was lucky enough to attend my first meetup in Poland this month: ![Agile Poznan](https://www.meetup.com/Agile-Poznan/events/242364938/). The talk was on ![Systems Thinking](http://managedagile.com/what-is-systems-thinking-and-why-is-it-important/), a topic that bored me in college but fascinated me when I entered the real world. Applying systems thinking can reveal the origins behind events and behaviors we see every day. Using this clarity can make you a better software developer, and a better asset to your organisation in general.  

Here are some of my insights from the meetup.

## A quick definition

A **system** is made up of elements that share a purpose. **Elements** can be people, machines, or other sub-systems within the system. The elements are connected within the **boundary** of a system.

You can identify a system through its elements and the connections between them, its purpose, and its boundary. Identifying the elements is easy. The purpose, boundary, and connections might be a little harder to find.

## It's not the fault of the person. It's the system.

A surprising majority of problems can be mitigated when you stop blaming the person that made a mistake, and build your system so that the problem won't happen again. The person is just the messenger that's showing you that the house is burning down.

Code reviews used to occasionally frustrate me before I came to this way of thinking. I'd find myself reiterating the same comments repeatedly. I want to learn, and I want to help others to learn, but learning gets diluted when we are forced to focus on miniscule issues like styling in a code review and miss the real opportunities for growth.

I see using a tool like ![Danger](http://danger.systems/ruby/), or adding any linting or code formatting to your development pipeline, as a great application of systems thinking. You're building safeguards in the system that will allow you to focus on more important and nuanced issues.

## The performance of an organisation depends on the quality of systems, not of individuals.

The overhead of trying to micromanage every individual in an organisation outweighs the possible benefits. You simply can't maintain it as the organisation grows and grows. The people are indispensable, but they come and go in a way that can't be controlled. What can be controlled is the systems that the organisation runs on.

When an organisation improves its systems, the benefit is increased productivity. But it's not just increased productivity right now, with the current teams and employees. It's a benefit for the people there now, and those that are still to come, whether in six months or six years. Spend the initial time up front setting up a system that fosters growth, and you'll save time and create value in the long run.

Here are some system improvements that could be used to improve company-wide performance:
- Allocate a self-education budget to each employee
- Issue a training manual to each new hire
- Organise office space with focus and productivity in mind (i.e. avoid open plan offices)

## Problems faced by an organisation can most likely be solved by a system.

It can be surprising to realise that most problems can be tied back to a system. Sometimes the system doesn't exist yet. That's the problem. Here are some examples of problems that might not necessarily look like system problems at the first glance, as well as a possible solution that takes the system into account:

#### A high frequency of software defects
Are there adequate QA processes? Could a safer programming language or framework be justified?

#### Underskilled developers
Try arranging weekly lightning talks related to the development that you're doing. Chances are, there are senior developers that would love the experience and junior developers that would love to learn.

#### The same bugs occurring repeatedly
You fix it one day, and then it's back the next. A company policy of writing a quick internal knowledge base article describing the cause, symptoms, and solution of the bug might help. (Two software bugs are rarely the same, and I can see this technique working better for something like tech support. However, if sharing your knowledge is what it takes to prevent yet another off-by-one error, then so be it.)

## Boundaries are flexible.

Not all problems fall within the bounds of your system. How then are we meant to apply systems thinking? It's still possible by *expanding the system*. One example is an app development shop that constantly deals with clients bringing in new requirements at the last minute. How can we prevent this? The client is external to the system. Changing our internal operating procedures won't help here.

One approach is to expand the system to include the client as an internal element, not an external problem. Include them as part of your internal processes. Let them be a part of designing the end product. Ask for their feedback often. (Sounds a lot like Agile, doesn't it?)

## Fix the problem, not the symptom.

If you have an aching back, a painkiller is a great remedy. Really does the trick, right? The pain goes away in no time. The problem is when the pain comes back the next day. And next month. And again, and again. 

We know not to only focus on symptoms. Get a bed that's better suited for your back. Go for walks and get some exercise. Change sitting positions often. 

The same thing can happen with software projects. When one is falling behind, we double down, add more manpower, and put our effort into shipping on time. Sometimes, it even works. Why, though, does it keep happening with all the subsequent projects?

Our focus shouldn't be on the currently failing project. It's just a symptom of a wider cause - a failing system. The problem doesn't lie in the current project that's executing, the problem lies in the system of how you execute software projects. Step back to see the wider system and its purpose, and you can apply a solution that'll work for the current problem, as well as the next one.

# A final thought

A surface level understanding and adoption of systems thinking provides a shift in perception that has proven beneficial to me. While you could do a series of lectures on the nuances and trade-offs against other methodologies, that's not what I want from it. A wider understanding of my surroundings and what is under my control, and how to affect it, is enough for me. 

Steven Covey paints a picture of the difference between managers and leaders in his book, The Seven Habits of Highly Effective People:
> Envision a group of producers cutting their way through the jungle with machetes. 
> The managers are behind them, sharpening their machetes, writing policy and procedure manuals, holding muscle development programs, bringing in improved technologies, and setting up working schedules and compensation programs for machete wielders.
> The leader is the one who climbs the tallest tree, surveys the entire situation, and yells, “Wrong jungle!” 

I'm certain that the leader in this story was a strong systems thinker.
