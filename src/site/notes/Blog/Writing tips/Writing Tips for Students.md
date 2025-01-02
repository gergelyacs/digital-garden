---
{"dg-publish":true,"permalink":"/blog/writing-tips/writing-tips-for-students/","created":"2025-01-03T00:09:10.545+01:00","updated":"2025-01-03T00:12:28.721+01:00"}
---


# Writing Tips for Students

Below you find a suggested structure of your technical work (for project lab reports, theses, and research papers) with some hints and general writing guidelines. It is not intended to be a complete guide, nor a universal structure that fits for all works. Some works may require a different structure (e.g., surveys or literature reviews). This guideline does not replace your advisor; you can only learn writing by practice, and you will definitely need the feedback on your writing. (Feedback _should_ be a sign of caring, even if it is negative. Only worry if you have not received any feedback.)

---
# Abstract

**Goal:** Summarize your work! It is essentially a summary of your introduction (Section 1) and conclusions (Section 7).

1. Motivation, problem (ca. 1/3 of the abstract) 
2. Goal of your work, perhaps with the idea of the solution
3. Main results

# 1. Introduction

**Goal:** “Sell your work"! Engage the reader!

- _Motivation:_ Why is the topic/problem important? Give pointers to real-world examples (non-scientific literature is also fine such as tech news). Convince the reader (and yourself) that your work is important/useful. Stay focused: don't start with common knowledge (such as importance of machine learning), start directly with the negative impact of its security problems if that's the (broader) topic.
  
- _Problem definition:_ What do you want to solve exactly? Your description should be less formal and rather widely accessible. Avoid formulas here, if possible.
  
- _Challenges_: Argue that the problem is challenging and focus on those challenges that you will address (and others have failed to solve). You can cite most related work if necessary. Provide the selling point (the gap in state-of-the-art that you are trying to fill with your work)!
  
- _The idea of your solution:_ Describe how you solve the above challenges/problem. Focus on intuition that helps the reader grasp the concept of your proposal, and defer technical details to Section 5. You should provide intuitive insights and the thinking that went behind your solution.  [Keep it simple](http://www.paulgraham.com/simply.html): remember that the point is to reach as many people as possible here and you probably need to (over)simplify/generalize claims that you will make precise later. This may seem overselling, but it is [necessary for effective communication](https://www.youtube.com/watch?v=XFqn3uy238E). If your approach consists of several steps and seems complicated at first, make a single illustrative figure that is preferably self-contained and summarizes your method. For example, you can decompose your method into smaller steps and make a box per step with an illustrative graphic. It will guide the reader through the rest of the paper, and can also be recycled later in talks (even by others). You can detail each step in subsequent sections.  
      
    
- _Specific contributions:_ List your contributions explicitly (2-3 bullet points, should not be more than 2-3 sentences per bullet point)
  
- _Structure of your work:_ Optionally describe the structure of your work (very shortly), if your work is long (for a thesis).

# 2. Related work

**Goal:** Describe and cite prior works which have addressed the _same problem_ as yours! Keep it focused!

- _Outline_ the difference between these works and yours. Why is your work better/different than others?
  
- “Prove" these claims in the evaluation/analysis section.

# 3. Background

**Goal:** Describe all material needed to understand your solution (e.g., convnets, LSTM, etc.)!

- The description of every related work and background material should be tailored to your work. For example, most people know what convnet is but they may not know why you use it. Highlight those properties of the tool, which make it appealing to solve your problem, and ignore the rest. Fit the description to your work and don't stick to the original text! Provide your own interpretation: This will give a new (your) viewpoint of the background material and hence bring value for the reader!
  
- Keep it focused, and don't write too much about textbook material if the target audience is likely to know that! Rather cite them! This is not the place where you should “inflate” your work. Shorter summaries are generally preferred over copy-pasted literature reviews. Surveys can be cited.

# 4. Model

**Goal:** Define the assumptions of your proposal! In what conditions can it be used? What does it need to work?

- _System model:_ Assumptions about your system: e.g., availability of training data, type of data, any constraints, etc.
  
- _Attacker/Adversary/Threat model:_ Assumptions about the adversary: its goals, capabilities, and the specific attacks (threats) used to achieve its goal. Argue about the feasibility and practicability of the attack. For example, the adversary has enough incentives to launch these attacks because there is a lot of gain or because the attacks are feasible and accurate.

# 5. Solution

**Goal:** Provide all details of your solution! Be precise, clear, and easy to follow! Present analyses after this section or in the appendix!

- Make a short overview of your approach first. If it is necessary, you can make a formal, step-by-step (e.g., algorithmic) description in a separate plot that you will detail in later subsections, and make your subsections according to this overview. At the beginning of each subsection, present shortly what you will do in that subsection (this links different (sub)sections together and makes your work easier to follow).
  
- Be precise, comprehensible and easy to follow. Use examples where possible, or refer to the illustrative plot in the introduction and to the more detailed algorithmic descriptions presented here (if you need such). Avoid long ambiguous sentences/paragraphs/phrases that the reader may not understand. Consistently use the same terms/words to refer to the same phenomena. 
  
- Focus on precise claims but do not distract the reader with overly detailed mathematical proofs or too many formulas. Defer detailed empirical proofs (evidence) to the evaluation section, and formal arguments/proofs to the end of this section ("analysis" (sub)section), or to the appendix. Every formal claim should also have an intuitive description first, neglecting formulas as much as possible. You do need proofs to be scientifically correct but remember that most people are not fluent in heavy maths and they want to have the insight first. Explain the notation in every formula!

# 6. Evaluation

**Goal:** Show that you solve the problem and compare it to related work empirically or analytically! "Empirical" usually means simulations and more detailed comparisons with the most relevant related works. "Analytical" means more rigorous mathematical reasoning (i.e., you have a formal model with premises and you derive some properties from these). The usefulness of such proofs mainly depends on how realistic your model (assumptions) are.

- _Dataset description:_ Describe your dataset (data collection, source of data, pre-processing, main descriptive statistics of your data) 
  
- _Baselines:_ Describe other approaches that you consider for comparison. You can describe them here or in the background section.
  
- _Parameter setting:_ Provide all necessary details to reproduce your results (hyperparameter setting, implementation environment with exact version numbers, etc.)! Make your code available to the reviewers! This will also earn you more citations.
  
- _Evaluation metrics:_ Metrics used to measure performance.
  
- _Results:_ Put results in a separate subsection! Highlight the main takeaways and conclusions!

# 7. Conclusion

- _Summary:_ summarize what you have done, its limits, why it is good/practical/etc.
  
- _Conclusions:_ messages (and not summary) of your work, takeaways
  
- _Future work:_ outline possible future directions (development, research, etc.)

# Appendix

- Proofs, extra evaluations, dataset descriptions (if there are many), etc.
- Most people will not read this, so put things here accordingly

---
# Further hints:

- Allocate enough time for writing! It's not only a soft skill, it also [improves critical thinking](http://www.paulgraham.com/words.html): when you write, you may find fallacies in your solution. Writing is an iterative process and takes time; you may re-write some parts of your work several times until you find its most accessible form. Start writing when you have shown that your solution works and obtained the first positive results. If you have negative results or get stuck, don't panic! Step back and start summarizing what you are trying to solve (focus on Sections 4 and 5). You can also put your problem aside for a while, which can help discover totally new approaches when you return to it later. If your results eventually turn out to be negative and you don't have time to try another approach (or you believe that your problem can't be solved), then write this and recalibrate your work! Your negative findings may save precious time for others only if you publish them.  
- Keep everything simple including maths and [writing](http://www.paulgraham.com/simply.html) in general:   
  
  _"The easier something is to read, the more deeply readers will engage with it. The less energy they expend on your prose, the more they'll have left for your ideas. ... fancy writing doesn't just conceal ideas. It can also conceal the lack of them. That's why some people write that way, to conceal the fact that they have nothing to say. Whereas writing simply keeps you honest. If you say nothing simply, it will be obvious to everyone, including you." (P. Graham)_  
  
  You are probably done when you doubt that your work deserve publication because your contributions seem so straightforward that anybody could have done what you did, still you provided every detail of your solution.    
- Be succinct (but precise)! Don't care about recommended minimum size limits, rather set a (reasonably small) maximum for yourself! It helps with focusing on the main points. Explaining something shortly but also clearly is challenging and has a positive impact on thinking and communication: you learn to be more direct and intelligible. [Technical communication](https://ohiostate.pressbooks.pub/feptechcomm/chapter/3-writing-style/) prioritizes the _efficient transfer of information_ - this can be quite different from “High school writing” that you might have get used to. A technical document is not a descriptive expository essay. To make your message unambiguous, use short and simple sentences and communicate only one concept per sentence. In technical writing, you document information and communicate it in a concise, precise, and professional way. _The focus is not to prove that you read or understand something_ but to describe/explain something as clearly and simply as possible. Be the teacher and not the student!   
- Avoid copy-pasted plots or texts, rather prepare them by yourself (at least the overview plot in the introduction)! Ownership gives you confidence and enthusiasm: you will be much more confident when you present your own work/plots making your talks more enjoyable. Every person has different background or even thinking, follow yours but prepare for the audience when you present!   
- Don't start with Section 1 (Introduction)! You can only sell your work if you already know its limitations and strong points, that is, you have finished Section 4, 5, 6. Some people start with the abstract or conclusion which helps with clarifying the goals before writing, others start describing their work directly (Section 4, 5 followed by Section 6).   
- Most people will only read the Abstract, Introduction, and Conclusion, and much fewer will read the rest. Take special care to write these parts, especially the Introduction that gives the first impression to the reader (and the reviewer)!