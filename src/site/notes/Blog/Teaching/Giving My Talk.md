---
{"dg-publish":true,"dg-path":"Teaching/Giving My Talk.md","permalink":"/teaching/giving-my-talk/","created":"2026-02-20T13:28:29.632+01:00","updated":"2026-02-27T00:09:17.101+01:00"}
---

Mass education in universities has many drawbacks, and one of the most significant is that students rarely get enough opportunities to develop soft skills - writing, speaking, and collaborating - even though these are crucial skills in practice. This post is my attempt to collect some observations from the past years. Just as with [[Blog/Teaching/Writing My Technical Paper\|writing hints]], these are subjective guidelines, not hard rules, and the only way to improve is to practice.

---
# Why Give a Talk at All?

Before thinking about how to give a good talk, it's worth asking why you're giving one. The reasons matter more than you might think.

For students, the most basic (and most limiting) reason is simply to defend their work. This is partly why I dislike the term 'defense.' It easily leads to a talk focused on proving knowledge rather than communicating it, which is the wrong instinct entirely. The audience doesn't want to be impressed; they want to understand and learn. A better goal is to motivate people, to share something you find genuinely interesting and make them feel the same way, or at the very least to make them want to read your paper.

Beyond that, there are less visible motivations to give a talk, the preparation is being one of them. I often accept invitations simply to force myself to read and understand something I've been putting off. Preparing a talk is one of the best ways to learn, and for me, that alone is reason enough.

This is because giving a talk deepens your understanding. There are roughly four levels: you read something and think you understand it (you don't); you can solve exercises and think you really understand it (you still don't); you can explain it to an expert (getting closer); and finally, you can explain it to a complete layman or a five-year-old. Even then, you probably don't fully understand it, but that's the point. Each new viewpoint is a new angle on the same problem, and the process of finding explanations across all of them is itself what deepens understanding.

Finally, giving talks improves your general ability to communicate. If you can explain something clearly to a hundred people, you'll explain it more clearly to one.

---

# It Starts with You, Not the Audience

Here is something that took me a while to internalize: the talk is more about you than the audience. You have to enjoy your own talk first. If you don't, the audience won't either. So the first question to ask yourself is not "what does the audience want to hear?" but "what am I genuinely enthusiastic about?"

Enthusiasm is contagious, but it has to be real. People tend to be more enthusiastic when they feel ownership over some part of what they're presenting - the problem, the motivation, or the solution - or when they just find the work really interesting. If you're presenting someone else's work without caring much about it, that will show. Find the angle that makes you care, spend more time on the parts that you find more interesting.

This also means acting like a teacher, not a student. Many students have been conditioned by oral exams and unconsciously treat the audience as a committee that already knows the material and is judging them. That's usually wrong. You are the one who knows your talk best. The audience is there to learn from you, not to evaluate you. Your job is to _explain_, not to demonstrate how much you know. 

---

# Self-Worth and Self-Confidence

Most advice about public speaking focuses on technique; slow down, speak up, make eye contact, don't read off your slides. All of that is useful. But there's a deeper issue that technique alone won't fix: how you relate to the talk itself, and what you think it says about you.

Self-confidence is crucial for a good talk, and it can be earned through mastery and experience. But if your self-worth is already tied to how well your presentation is received, you're in trouble before you even open your mouth. You cannot fully control the audience - someone might ask a tough question, or people might seem disengaged - and tying your self-worth to things you cannot control means living under a constant threat, one that quietly kills your confidence and eventually the talk itself. The role of 'expert presenter' can be taken away at any time. Your curiosity and your genuine interest in the problem cannot. Build your identity (and your talk) around those, not around being impressive.

This requires accepting your weaknesses honestly. Every presenter has specific, real ones - rushing through slides, over-explaining, avoiding eye contact. The temptation is to either ignore them or feel guilty about them. Neither helps. The productive move is to name them clearly, accept that they exist, and work on them one at a time. You don't have to fix everything at once, and some things may never be fully fixed, and that's fine too.

And don't let negative labels stick, whether they come from yourself or from others. People will put you in boxes no matter what you do. Once you believe a label, you behave accordingly, and the belief confirms itself. Treat your current limitations as habits, not traits - things that came from somewhere and can be changed with effort. And if they can't, accept them and move on. I've attended brilliant talks given by people who were shy, soft-spoken, fast-talking, or slow. Style matters far less than substance and genuine engagement.

---

# Know Your Audience

A talk that works brilliantly for experts will fail completely for a general audience, and vice versa. This is one of the most common mistakes.

The key concept here is intuition, which is finding the shortest explanation of the key idea that everyone in the audience can follow, and that connects most naturally to their existing knowledge. If the audience is diverse, that explanation needs to be universally accessible. If you can explain it so that a layman understands, you can adapt to any audience. And if you can't, then perhaps even you don't fully understand it yourself.

One thing is always true regardless of audience: people are lazy. Not in a bad way - they've attended many talks before yours, they're tired, and they have limited attention. You have to make your message simple and intuitive, especially if you're speaking late in the day or last in a long session. Assume less prior knowledge than you think is necessary, and simplify further.

---

# What is the Intuition?

A useful way to think about intuition is through the lens of a knowledge graph. Imagine that everything a person understands is a node in a graph, and understanding means being well-connected; the more links a concept has to other concepts, the more robustly it is understood. By this view, intuition is the simplest explanation that maximizes connectivity. A formal definition or an equation may be technically precise, but it often connects to very few existing nodes for most audiences, and what is weakly linked is quickly forgotten. A good intuition, by contrast, borrows the connectivity of something the audience already knows well, showing that the new concept is really just a familiar idea in disguise.

Take linear algebra as an example. Almost every concept in it has a clean geometric interpretation accessible to nearly anyone. A matrix is not just a grid of numbers, it is a transformation of space. Matrix multiplication is simply applying two transformations in sequence. The determinant measures how much the transformation stretches or squashes space; it is the area of the parallelogram formed by the column vectors in 2D, or the volume of the parallelepiped in 3D. A determinant of zero means the parallelogram has collapsed to a line: the vectors are linearly dependent, information is lost, and the matrix cannot be inverted. The null space is the set of directions that get squashed to zero, and the rank is how many dimensions survive. Eigenvectors are the directions that don't rotate under a transformation, they only stretch or shrink, forming the natural axes along which the transformation acts most simply, with the eigenvalue as the stretching factor. Repeatedly applying the same transformation drives any vector toward the direction of the eigenvector with the largest eigenvalue, with its magnitude growing as the power of this largest eigenvalue (this is the principle behind Google's PageRank). The dot product measures the shadow of one vector onto another, capturing how much they point in the same direction, which is why it appears everywhere from similarity measures to covariance in probability. Even Singular Value Decomposition (SVD), which can seem quite abstract, is in fact just a rotation, followed by a scaling, followed by another rotation - much like prime factorization reduces any integer to its simplest components. This geometric viewpoint also makes more advanced topics that build on linear algebra, such as Markov chain theory or machine learning, more coherent and accessible.

The knowledge graph analogy also explains the levels of understanding described earlier: each new viewpoint adds new connections. The more clusters you can link a concept to, the more robustly it is represented and the more naturally it will be recalled and applied later. So the practical question when preparing a talk is not 'how do I explain this correctly?' but rather: what does this audience's knowledge graph already look like, and which path through it leads most naturally to what I want them to understand? It's about communication in the first place, and then about correctness.

---
# More is Sometimes Less

This is particularly true for talks, though it applies to [[Blog/Teaching/Writing My Technical Paper\|writing]] as well. I often find myself getting excited about a topic and wandering into rabbit holes; side details, edge cases, subtleties that matter to me but lose the audience. The enthusiasm is genuine, but it works against communication.

Interestingly, people who know less about a topic often present it better. A supervisor who only understands the essentials can sometimes give a clearer overview than the student who built the whole system. They haven't seen every dead end and aren't tempted to explain them. They understand only what needs to be understood, and that's exactly what they convey.

Ask yourself: what is the single sentence that summarizes this talk? What is the one thing I want the audience to take away? If you can't answer that clearly, the talk isn't ready yet.

---

# Structure: Motivation First, Intuition Over Formalism

A talk and a [[Blog/Teaching/Writing My Technical Paper\|paper]] have a similar structure, but the emphasis is different. In a talk, the motivation and the problem should take up almost a third - sometimes even half - of the time, ideally with real-life examples or demos.

A common mistake, especially among students, is to rush through this part or skip it almost entirely, assuming the audience already knows why the problem matters and is just waiting for the solution. This is almost always wrong. Even if they know the problem, they might not have your viewpoint on that.

Motivation means explaining why the problem exists in the first place, preferably through a concrete, relatable example, and what the real impact would be if it remained unsolved, whether socially, economically, or technically. This part should be emotionally compelling rather than precise. 

The problem definition that follows should be more (but not overly) precise, laying out the model and the assumptions clearly. Together, motivation and problem definition set up the goal of your talk, and the audience needs enough _time_ to digest and memorize both. Remember that they may have just come from a completely different talk and need a moment to switch context. You have to be sure that by the time you reach your solution, everyone in the room understands and remembers the problem well.

This matters because the solution is likely to be missed anyway if it is too technical. But if the motivation and the problem are not crystal clear to everyone in the room, the whole talk is in vain. Your job is to convey the intuition behind the solution: what is the core idea, and why does it work?

Equations are a particular danger. Many people automatically switch off the moment they see a formula, without even giving the presenter a chance. If you must include one, don't spend time on derivation, just explain what it says. What is the intuition? What does this expression want to communicate? That single sentence (or a well-placed illustration) is worth more than introducing useless notation that nobody will remember by the next slide.

Finally, don't confuse a summary with a takeaway. A summary recaps what you said. A takeaway tells the audience what to do with it - the potential impact, the open question, the thing worth remembering. After a ten-minute talk, a summary may be unnecessary. A good takeaway never is.

---

# Format

The ends justify the means here - as long as your format help the audience understand, anything goes, and anything that makes your message harder to receive is a problem. This includes distracting backgrounds, logos and status bars that compete for attention, fonts that fit the template's style yet are hard to read from the back of the room, and insufficient contrast between text and background. Always include slide numbers so people can refer back during questions. When you reach the Q&A, leave your conclusion slide on screen rather than returning to a title slide or blank - it keeps the key message visible while people are thinking.

People have limited focusing capacity, they are either listening to you or reading your slides, rarely both at once. Too much text and too many distracting elements pull their attention away from what you're saying, so simple images, diagrams, and animations are generally preferable. But keep transitions and animations minimal; flashy effects draw attention to themselves rather than to your content. If you use code, equations, or dense figures, ask yourself whether a simpler version would do the job just as well, it usually would. And avoid putting anything on a slide you don't intend to address: if it's there, people will read it stealing their attention for no purpose.

Remember that what matters most is what you say, not what's written behind you, hence the word 'talk'. Your slides should support you, not replace you. Again, focus on yourself, and not the audience; if your slides help you tell the story clearly, they will help the audience too.

---

# Storytelling

People are [naturally drawn to stories](https://perceiving-systems.blog/en/post/writing-a-good-scientific-paper). The motivation section is the most natural place for storytelling, but it can appear anywhere in a talk. When I notice I'm losing the audience  - eyes glazing over, phones appearing - I try to come back to a concrete story or example. It almost always helps.

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

For a long time I assumed I had some deficiency because I could barely follow technical talks as an undergraduate, and for a while even after that. It wasn't until I attended a genuinely good talk - one where the speaker took special care to build intuition before formalism - that I realized what had been missing. A good talk is not about technical soundness. It's about communication. The speaker's job is to simplify, sometimes at the cost of precision, in order to hold attention and motivate the audience to go read the actual paper.

Equally important: record yourself and watch it back. It's uncomfortable precisely because it closes the gap between how you imagine you come across and how you actually do. Painful and sometimes embarrassing, but you can't improve what you can't see, and the recording shows you exactly what to fix.

---

# What Can You Lose?

People often say you can't lose with a bad talk. That's mostly true for a single talk, since audiences forget quickly, and in a conference setting, your paper matters far more than your presentation. Even in a thesis defense, the reviewers' judgment of the written work is what counts.

What you can lose is being remembered. A good talk makes people remember you; a bad one means they forget you faster. What really matters is consistency - if you consistently deliver poor talks as a teacher, that accumulates. A single bad talk is nothing to worry about; the variance in quality is part of being human. As Maxwell Maltz put it: _"We make mistakes, mistakes don't make us."_ A bad talk is something that happened. It is not who you are. The goal is simply to notice when you're consistently underperforming and do something about it.

Giving a talk is a lot like cooking. Sometimes it comes out well, sometimes it doesn't, and the reasons are often beyond your control. A bad night's sleep, a personal problem, technical difficulties - all of these affect the result in ways that no amount of preparation can fully neutralize. The honest thing to do is accept this rather than pretend otherwise. Most food is enjoyable enough when people are hungry, and most talks land well enough when the audience really wants to learn something. 

---

# Final Note

Don't get too caught up in the nuances - the balance of text and figures, eye contact, timing, font sizes, slide layouts, whether you thank the audience on the final slide or not. These details can help at the margin, but none of them is what makes or breaks a talk.

What matters is what you say, and whether you care about it. Focus on the intuition. Simplify the message as much as possible. And remember: enjoy your own talk first. Only then can the audience enjoy it too.

