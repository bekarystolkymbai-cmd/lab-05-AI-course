# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 changed my results the most: from 16 correct / 4 wrong / 4 no answer to 18 / 6 / 0 (out of 24 — the set has 24 questions, not 20). It changed what the agent can do, not what it admits. Before, a tool error stopped the whole run and gave nothing; after, the model saw the error and could try again. All four former non-answers got an answer: two alias questions (q23, q24) became correct, but q21 and q22 became wrong answers instead of nothing. So the agent got more capable and also bolder — some honest “no answer”s turned into confident mistakes. With 24 questions, a gain of two is small. Layer 2 changed nothing: 18 / 6 / 0 and the same 48708 tokens as layer 1.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

I pick q08: “which region had the lowest average monthly wage in 2024”. Layer 3 still answers Jambyl, which is wrong — the correct answer is [X]. The tool step was fine: the model called compare_regions with indicator avg_monthly_wage and year 2024, and the tool returned all regions sorted highest first, Astana on top and [X] at the bottom. The error is in the next step: the model read the list and wrote “ANSWER: Jambyl” instead of taking the last row. The harness did not catch it because grounding only checks numbers; “Jambyl” really appears in the tool output, so the answer looks grounded. I would add a code check for rank questions: if the question asks for the lowest or highest, compare the answer with the region that has the min or max value in the tool output, and if they differ, return no answer. This costs no extra model calls, almost no tokens and only a few lines of code, but it turns q08 into an honest “no answer” rather than a correct one; asking the model to retry would fix it but adds one model call. New ways to fail: the check must recognise rank questions from words like “lowest” or “highest”, and it can miss other wordings; and a strict name comparison could reject a correct answer with a different spelling, which the existing alias normalisation should handle.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

For a monthly ranking I would ship the workflow. On the 15 fixed questions it got 13/15 correct with 15 model calls, 3852 tokens and 15.9 seconds; the bare agent got 9/15 with 26 calls, 19835 tokens and 50.9 seconds. So the workflow was more accurate and used about 5× fewer tokens and 3× less time. The sample is only 15 questions, but the gap is large. The ranking question always has the same shape, so a fixed pipeline is enough, and with exactly one model call per question its monthly cost and output are predictable and easy to check before publishing. I would choose the agent only when the question can change shape, for example a growth or percentage-point question where the model must decide which tools to call and in what order — on such open questions the bare agent still got 7/9.

---

I wrote this file myself, without any AI tool.
