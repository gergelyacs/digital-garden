---
{"dg-publish":true,"dg-path":"Teaching/Giving My Talk.md","permalink":"/teaching/giving-my-talk/","created":"2026-02-20T13:28:29.632+01:00","updated":"2026-03-21T09:21:45.482+01:00"}
---

Mass education in universities has many drawbacks, and one of the most significant is that students rarely get enough opportunities to develop soft skills - writing, speaking, and collaborating - even though these are crucial in practice. This post is my attempt to collect some observations from the past years. Just as with [[Blog/Teaching/Writing My Technical Paper\|writing hints]], these are subjective guidelines, not hard rules, and the only way to improve is to practice.

---
# Why Give a Talk at All?

Before thinking about how to give a good talk, it's worth asking why you're giving one. The reasons matter more than you might think.

For students, the most basic (and most limiting) reason is simply to defend their work. This is partly why I dislike the term 'defense.' It easily leads to a talk focused on proving knowledge rather than communicating it, which is the wrong instinct entirely. The audience doesn't want to be impressed; they want to understand and learn. A better goal is to motivate people, to share something you find genuinely interesting and make them feel the same way, or at the very least to make them want to read your paper.

Beyond that, there are less visible motivations to give a talk, the preparation is being one of them. I often accept invitations simply to force myself to read and understand something I've been putting off. Preparing a talk is one of the best ways to learn, and for me, that alone is reason enough.

This is because giving a talk deepens your understanding. There are roughly four levels: you read something and think you understand it (you don't); you can solve exercises and think you really understand it (you still don't); you can explain it to an expert (getting closer); and finally, you can explain it to a complete layman or a five-year-old. Even then, you probably don't fully understand it, but that's the point. Each new viewpoint is a new angle on the same problem, and the process of finding explanations across all of them is itself what deepens understanding.

Finally, giving talks improves your general ability to communicate. If you can explain something clearly to a hundred people, you'll explain it more clearly to one.

---

# It Starts with You, Not the Audience

Here is something that took me a while to really understand: _the talk is more about you than the audience_. You have to enjoy your own talk first. If you don't, the audience won't either. So the first question to ask yourself is not "what does the audience want to hear?" but "what am I genuinely enthusiastic about?"

Enthusiasm is contagious, but it has to be real. People tend to be more enthusiastic when they feel ownership over some part of what they're presenting - the problem, the motivation, or the solution - or when they just find the work really interesting. If you're presenting someone else's work without caring much about it, that will show. Find the angle that makes you care, spend more time on the parts that you find more interesting.

This also means acting like a teacher, not a student. Many students have been conditioned by oral exams and unconsciously treat the audience as a committee that already knows the material and is judging them. That's usually wrong. You are the one who knows your talk best. The audience is there to learn from you, not to evaluate you. Your job is to _explain_, not to demonstrate how much you know. 

---

# Self-Worth and Self-Confidence

Most advice about public speaking focuses on technique; slow down, speak up, make eye contact, don't read off your slides. All of that is useful. But there's a deeper issue that technique alone won't fix: how you relate to the talk itself, and what you think it says about you.

_Self-confidence_ is crucial for a good talk, and it can be earned through mastery and experience. But if your self-worth is already tied to how well your presentation is received, you have a problem before you even open your mouth. You cannot fully control the audience - people might be exhausted from previous talks, distracted, or asking tough questions - and tying your self-worth to things outside your control means living under a constant threat, one that quietly kills your confidence and eventually the talk itself. The role of 'expert presenter' can be taken away at any time. Your curiosity and your genuine interest in the problem cannot. Build your talk around those, not around being impressive.

This requires accepting your weaknesses honestly. Every presenter has specific, real ones - rushing through slides, over-explaining, avoiding eye contact. The temptation is to either ignore them or feel guilty about them. Neither helps. The productive move is to name them clearly, accept that they exist, and work on them one at a time. You don't have to fix everything at once, and some things may never be fully fixed, and that's fine too.

And don't let negative labels stick, whether they come from yourself or from others. People will put you in boxes no matter what you do. Once you believe a label, you behave accordingly, and the belief confirms itself. Treat your current limitations as habits, not traits - things that came from somewhere and can be changed with effort. And if they can't, accept them and move on. I've attended brilliant talks given by people who were shy, soft-spoken, fast-talking, or slow. Style matters far less than substance and genuine engagement.

---

# Know Your Audience

A talk that works well for experts can fail completely for a general audience, and vice versa. This is one of the most common mistakes.

The key concept here is _intuition_, which is finding the shortest explanation of the key idea that everyone in the audience can follow, and that connects most naturally to their existing knowledge. If the audience is diverse, that explanation needs to be universally accessible. If you can explain it so that a layman understands, you can adapt to any audience. And if you can't, then perhaps even you don't fully understand it yourself.

One thing is always true regardless of audience: people are lazy. Not in a bad way - they've attended many talks before yours, they're tired, and they have limited attention. You have to make your message simple and intuitive, especially if you're speaking late in the day or last in a long session. Assume less prior knowledge than you think is necessary, and simplify further.

---

# What is the Intuition?

A useful way to think about intuition is through the lens of a knowledge graph. Imagine that everything a person understands is a node in a graph, and understanding means being well-connected; the more links a concept has to other concepts, the more robustly it is understood. By this view, intuition is the simplest explanation that maximizes connectivity. A formal definition or an equation may be technically precise, but it often connects to few central nodes in most audiences' graphs, and what is weakly linked is quickly forgotten. A good intuition, by contrast, borrows the connectivity of something the audience already knows well, showing that the new concept is really just a familiar idea in disguise.

Take linear algebra as an example. Almost every concept in it has a clean geometric interpretation accessible to nearly anyone. A matrix is not just a grid of numbers; it is a transformation of space, and matrix multiplication is simply applying two such transformations in sequence. In fact, any linear transformation can be decomposed into just three steps: a rotation, a scaling, and another rotation, much like prime factorization reduces any integer to its simplest components. This is precisely what SVD tells us. The determinant measures how much the transformation stretches or squashes space; it is the area of the parallelogram formed by the column vectors in 2D. A determinant of zero means the parallelogram has collapsed to a line: the vectors are linearly dependent, information is lost, and the matrix cannot be inverted. The null space is the set of directions that get squashed to zero, and the rank is how many dimensions survive. The dot product measures the shadow of one vector onto another, capturing how much they point in the same direction, which is why it appears everywhere from similarity measures to covariance in probability. And so on.

This geometric viewpoint also makes more advanced topics that build on linear algebra, such as machine learning, more coherent and accessible. The more clusters you can link a concept to in your knowledge graph, the more robustly it is represented and the more naturally it will be recalled and applied later. 

---
# Communication comes first, correctness follows

Time constraints require simplification, and simplification has a cost: the talk will not be fully precise. But that is the wrong thing to optimize for. Analogies and intuitions are never completely faithful, they deliberately sacrifice detail in order to keep the problem accessible. But it's better to draw in as many people as possible than to lose them by sacrificing clarity on the altar of precision.

The purpose is not completeness but to motivate the audience to discover the full details themselves. So the practical question when preparing a talk is not 'how do I explain this correctly?' but rather: what does this audience's knowledge graph already look like, and which path through it leads most naturally to what I want them to understand? A good talk is not about technical soundness, it's about communication in the first place. The speaker's job is to simplify, sometimes at the cost of precision, in order to hold attention and motivate the audience to go and read the actual paper.

---

# More is Sometimes Less

This is particularly true for talks, though it applies to [[Blog/Teaching/Writing My Technical Paper\|writing]] as well. I often find myself getting excited about a topic and wandering into rabbit holes; side details, edge cases that matter to me but lose the audience. The enthusiasm is real, but it works against communication.

Interestingly, people who know less about a topic often present it better. A supervisor who only understands the essentials can sometimes give a clearer overview than the student who built the whole system. They haven't seen every dead end and aren't tempted to explain them. They understand only what needs to be understood, and that's exactly what they convey.

Ask yourself: _what is the single sentence that summarizes this talk?_ What is the one thing I want the audience to take away? If you can't answer that clearly, the talk isn't ready yet.

---

# Structure: Motivation First, Intuition Over Formalism

A talk and a [[Blog/Teaching/Writing My Technical Paper\|paper]] have a similar structure, but the emphasis is different. In a talk, the motivation and the problem should take up almost a third of the time, ideally with real-life examples or demos.

A common mistake, especially among students, is to rush through this part or skip it almost entirely, assuming the audience already knows why the problem matters and is just waiting for the solution. This is almost always wrong. Even if they know the problem, they might not have your viewpoint on that.

Motivation means explaining why the problem exists in the first place, preferably through a concrete, relatable example, and what the real impact would be if it remained unsolved, whether socially, economically, or technically. This part should be emotionally compelling rather than precise. The problem definition that follows should be more (but not overly) precise, laying out the model and the assumptions clearly.

Together, motivation and problem definition set up the goal of your talk, and the audience needs enough _time_ to digest and memorize both. Remember that they may have just come from a completely different talk and need a moment to switch context. You have to be sure that by the time you reach your solution, everyone in the room understands and remembers the problem well. This matters because the solution is likely to be missed anyway if it is too technical. But if the motivation and the problem are not crystal clear to everyone in the room, the whole talk is in vain. Your job is to convey the intuition behind the solution: what is the core idea, and why does it work? 

Equations are a particular danger. Many people automatically switch off the moment they see a formula, without even giving the presenter a chance. If you must include one, don't spend time on derivation, just explain what it says. What is the intuition? What does this expression want to communicate? That single sentence (or a well-placed illustration) is worth more than introducing useless notation that nobody will remember by the next slide.

Finally, don't confuse a summary with a takeaway. A summary recaps what you said. A takeaway tells the audience what to do with it; the potential impact, the open question, the thing worth remembering. After a ten-minute talk, a summary may be unnecessary. A good takeaway never is.

---

# An Example

Consider the problem of computing the importance of every webpage on the internet, roughly what Google does. The following is not a perfect introduction to PageRank, but it shows how a complex topic can be introduced gradually, each step adding new connections to the audience's knowledge graph. The motivation is straightforward:

**Motivation**: *We want to rank webpages by general importance so that when we retrieve pages relevant to a query, we can prioritize the most authoritative and trustworthy ones. This saves time and ultimately accelerates progress in every area of life. A natural follow-up question is: how do we determine the importance of a webpage? The naive answer - rank pages by the number of incoming links - favors quantity over quality. A page with many links from obscure, irrelevant sources would rank higher than a page with few links from authoritative ones. Worse, this would be trivially exploitable: adversaries could inflate rankings by creating large numbers of artificial pages pointing to their target.*

Note that we motivated the problem from several angles, usefulness as well as robustness, and showed that the naive solution fails. This hopefully awakens the audience's curiosity

**Problem:** *The insight is the same as in social networks: a few connections to popular, well-connected pages are worth far more than many connections to isolated ones, just as being friends with a popular person provides more social benefit than having many unpopular friends. This brings us to the actual problem: how do we assign importance scores that reflect the global structure of the web, where every page's score depends on every other page's score?*

You have to be sure that by the time you reach your solution, everyone in the room understands and remembers the problem well, which is why we returned to the social network analogy above.

For a general audience, you probably wouldn't open the solution with Markov chains. Instead, you might start with something like this: 

**Intuition of the solution:** *Think of it as deciding the most popular kid in school. Instead of just counting how many friends a kid has, we also care about how popular those friends are; being connected to a popular kid counts more than being connected to someone nobody knows. To measure popularity, we start by distributing a fixed number of tokens equally among all kids. Each round, every kid splits their tokens equally among their friends and passes them on. A kid accumulates more tokens if their friends are popular, since popular friends also accumulate more tokens to redistribute. As a result, the majority of tokens will circulate among the popular kids. After enough rounds, the number of tokens that each kid holds stabilizes; they receive exactly as many as they pass on. Remarkably, we end up with the same count per kid regardless of how the tokens were initially distributed. That stable token count is their final popularity ranking.*

This is an oversimplification, of course, and it skips the mathematical details that actually explain why the token counts stabilize. But that is fine. Dwelling on those details at this point risks losing the audience before they are motivated enough to care about them. For computer engineers, the above kid example generalizes naturally:

**Solution:** *Represent the web as a directed graph where importance is circular - a node is important if important nodes point to it. To resolve this chicken-and-egg problem, turn it into an iterative algorithm: start with equal scores, then repeatedly update each page's score as the weighted sum of the scores of pages pointing to it, where each contribution is divided by the number of outgoing links of the pointing page. Repeat until convergence.*

But why does this work, and why do the scores stabilize? 

**Analysis:** *The final popularity scores of all webpages form the stationary distribution of an ergodic Markov chain on the web graph: the long-run landing probability of a random surfer, regardless of where they start*. 

For many engineers, this is already enough. Even if some details are still fuzzy, they can now anchor webpage ranking as just another instance of a Markov chain. In a longer lecture, however, we can go deeper: 

*Think of all possible token distributions among the webpages as points in a large space, and the redistribution rule as a map $P$, where $P_{ij} = 1/d_j$ if page $j$ links to page $i$, with $d_j$ being the number of outgoing links of page $j$. Applying $P$ pushes each point to a new location - most distributions are not yet consistent with the web structure, so they keep moving. But one special point doesn't move at all: the fixed point $\pi$ satisfying $\pi = P\pi$. Every other point is pulled toward it like a ball rolling to the bottom of a bowl.*

Why does the bowl exist and why do we always end up at the same $\pi$ regardless of where we start? 

*Notice that $\pi = P \pi$ looks exactly like an eigenvector equation with eigenvalue 1: $\pi$ is a vector that is completely unchanged by the transformation $P$. Repeatedly applying $P$ to any vector drives it toward the dominant eigenvector of $P$, while all other components decay, since every column of $P$ is a probability distribution summing to 1. The full explanation follows from the eigendecomposition of $P$...*

From here, the mathematics gets heavier, so we can stop. The key is to go top-down rather than bottom-up by first targeting the most central and well-connected nodes in the audience's existing knowledge. Start with a concrete analogy, and only then build toward the formal details if time allows. Physical analogies like the token distribution or the rolling ball, and geometric intuitions like those behind linear algebra, are especially useful because they connect abstract concepts to familiar ones, making them easier to recall and apply later even in more complex topics. If you know your audience well, you can skip some steps in the top-down presentation to leave more time for the mathematics, but you always risk losing people, since audiences are never homogeneous. In any case, you will probably not have time to cover all the mathematics in a 15-minute talk.

**Conclusion:** *A good takeaway is that importance scores reflecting the global structure of any graph can be computed through a simple local rule applied iteratively, since each node only needs to know the scores of its neighbors, and the global structure emerges from the iteration. This opens the door to many practical generalizations. More broadly, it is an instance of a fundamental principle: global properties can emerge from local rules. This appears everywhere, from physics to machine learning.*

These observations are worth far more than a recap of the algorithm itself, especially in a short talk.

---

# Format

The ends justify the means here - as long as your format help the audience understand, anything goes, and anything that makes your message harder to receive is a problem. This includes distracting backgrounds, logos and status bars that compete for attention, fonts that fit the template's style yet are hard to read from the back of the room, and insufficient contrast between text and background. Always include slide numbers so people can refer back during questions. When you reach the Q&A, leave your conclusion slide on screen rather than returning to a title slide or blank - it keeps the key message visible while people are thinking.

People have limited focusing capacity, they are either listening to you or reading your slides, rarely both at once. Too much text and too many distracting elements pull their attention away from what you're saying, so simple images, diagrams, and animations are generally preferable. But keep transitions and animations minimal; flashy effects draw attention to themselves rather than to your content. If you use code, equations, or dense figures, ask yourself whether a simpler version would do the job just as well, it usually would. And avoid putting anything on a slide you don't intend to address: if it's there, people will read it stealing their attention for no purpose.

Remember that what matters most is what you say, not what's written behind you, hence the word 'talk'. Your slides should support you, not replace you. Again, focus on yourself, and not the audience; if your slides help you tell the story clearly, they will help the audience too.

---

# Storytelling

People are [naturally drawn to stories](https://perceiving-systems.blog/en/post/writing-a-good-scientific-paper). The motivation section is the most natural place for storytelling, but it can appear anywhere in a talk. When I notice I'm losing the audience  - eyes glazing over, phones appearing - I try to come back to a concrete story or example. It usually helps.

---

# Questions

Questions are the best implicit feedback a talk can receive. If people ask, it means they got engaged, and that is one of the main purposes of a talk. No questions, on the other hand, often indicates a lack of engagement, or that the audience lost the thread somewhere along the way. Some presenters deliberately leave out important details to provoke curiosity and draw the audience in. It's a risky move, but it can work.

That said, the absence of questions does not always mean the talk was bad. Some audiences don't ask out of respect; others because they fear looking stupid; others simply because they never got positive reinforcement for asking. Questions vary culturally too. In more collectivist cultures, people may not raise their hand, but their eyes will often tell you. 

As a presenter, creating an environment where questions feel safe is part of the job. A good starting point is to remember that there are no stupid questions; if someone is confused, it is almost always because something wasn't explained clearly enough, not because the person is slow. When you get a question, answer it directly. Sometimes the honest answer is "I don't know," and that is perfectly fine. Sometimes it's simply "yes" or "no," and that is fine too. 

---

# Practice

The importance of practicing out loud cannot be overstated, and it is completely different from practicing in your head. In high school I used to summarize lectures out loud to myself, and it worked surprisingly well. You are literally tuning your brain to speak.

A useful mindset shift: giving a talk to a hundred people is not fundamentally different from explaining something to one person face to face. You don't need to be more careful, more formal, or more perfect. People actually like causal presenters; it signals honesty and confidence. Think about how you'd explain it to a colleague over coffee, that's roughly what you should do in front of an audience. 

But no matter how many rehearsals you do, real confidence only comes from real experience. You become a confident presenter by presenting repeatedly, out loud, in front of real people. The physical experience of speaking, pausing, losing your thread and recovering, that's where real confidence is built. 

---

# Watch Good Talks, and Watch Yourself

For a long time I assumed I had some deficiency because I could barely follow technical talks as an undergraduate, and for a while even after that. It wasn't until I attended a genuinely good talk - one where the speaker took special care to build intuition before formalism - that I realized what had been missing. 

Equally important: record yourself and watch it back. It's uncomfortable precisely because it closes the gap between how you imagine you come across and how you actually do. Painful and sometimes embarrassing, but you can't improve what you can't see, and the recording shows you exactly what to fix.

---

# What Can You Lose?

People often say you can't lose with a bad talk. That's mostly true for a single talk, since audiences forget quickly, and in a conference setting, your paper matters far more than your presentation. Even in a thesis defense, the reviewers' judgment of the written work is what counts.

What you can lose is being remembered. A good talk makes people remember you; a bad one means they forget you faster. What really matters is consistency - if you consistently deliver poor talks as a teacher, that accumulates. A single bad talk is nothing to worry about; the variance in quality is part of being human. As Maxwell Maltz put it: _"We make mistakes, mistakes don't make us."_ A bad talk is something that happened. It is not who you are. The goal is simply to notice when you're consistently underperforming and do something about it.

Giving a talk is a lot like cooking. Sometimes it comes out well, sometimes it doesn't, and the reasons are often beyond your control. A bad night's sleep, a personal problem, technical difficulties - all of these affect the result in ways that no amount of preparation can fully neutralize. The honest thing to do is accept this rather than pretend otherwise. Most food is enjoyable enough when people are hungry, and most talks land well enough when the audience really wants to learn something. 

---

# Final Note

Don't get too caught up in the nuances - the balance of text and figures, eye contact, timing, font sizes, slide layouts. These details can help at the margin, but none of them is what makes or breaks a talk.

What matters is what you say, and whether you care about it. Focus on the intuition. Simplify the message as much as possible. And remember: enjoy your own talk first. Only then can the audience enjoy it too.

