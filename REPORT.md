# REPORT

This report is about the scholarship screening agent from Lab 06. The agent reads an application and says "shortlist" or "reject" and gives a score from 0 to 100. The model is Qwen2.5-3B-Instruct, prompt version v1, greedy decoding. I tested it on 200 matched pairs: 40 pairs for each of three attributes (name language, gender of the name, university tier) and 40 pairs for each of two floors (a different name of the same kind, a different university of the same tier).

All numbers in the AUTO blocks come from my notebook run. The notebook writes them into this file. I did not type them by hand. The rules for reading the numbers were written in the code before the run: a one-sided Fisher exact test for the flips (attribute against its floor) and a bootstrap interval for the difference in mean absolute score difference.

## 1. Measurement read-out

### 1.1 What the run measured

<!-- AUTO:MEASUREMENT:BEGIN -->
(filled in by the notebook run)
<!-- AUTO:MEASUREMENT:END -->

### 1.2 How I read it

A pair is the same applicant two times with one change. If the decision is different inside a pair, there are two possible reasons. Reason one: the attribute. Reason two: the model reacts to any small change in that field. The floor measures reason two. So I say an attribute changed the decision only when its flips or its score differences are bigger than the flips or score differences of its own floor. The verdict lines in 1.1 say this for each attribute.

If a verdict line says "does not stand clear of its floor", I can only say this: the attribute changed the outputs not more than a random name or university of the same kind. I cannot say "the agent is fair". The power table in 1.1 shows why. With only 40 pairs, the test finds a real effect of 10 percent of pairs (all in one direction) only a small part of the time. So a real effect of this size can easily stay hidden. To see smaller effects I need more pairs, not the same 40 pairs again. With greedy decoding the same pairs always give the same answers.

I also look at the absolute score difference, not only the signed one. The signed mean can be close to zero when some twins go up and some go down. When the floor has a score difference bigger than zero, this is also a finding: the score moves by some points when only an unimportant name or university changes. This is why I use a score band for human review and not one single cut-off.

What the floors do not test. The floors keep the prompt text, the order of the lines, the decoding and the GPU the same. So they do not tell me how much the model reacts to a new prompt text (not tested), to a new order of the lines (the order probe in 1.1 checks this), or to another GPU (not measured). There is one more limit. The attribute pairs and the floor pairs are different applicants. If one group has more applicants near the decision line, it will have more flips for reasons that have nothing to do with the attribute. A better test would use the same applicants in all five kinds of pairs, several versions of the prompt, and more pairs.

Where the result does not reach. All names and universities are invented. A real system would see real names and real universities, and the model learned things about them in training. There the effect can be bigger or smaller, and this run cannot say which. A new model or a new prompt is a new system, and it needs a new measurement. My condition of use is: run Part 1 again when the model, the prompt or the shown fields change. The stop checks in Part 3 of the notebook are only for this configuration.

## 2. Who is affected

### 2.1 What the run shows for each group

<!-- AUTO:AFFECTED:BEGIN -->
(filled in by the notebook run)
<!-- AUTO:AFFECTED:END -->

### 2.2 Who pays for a wrong decision

The applicant who is rejected by mistake. Every flip in 2.1 is one person whose result changed because of something that must not matter. In an attribute pair, it is the language or gender of the name, or the type of university. In a floor pair, it is nothing at all. A wrong rejection means no scholarship: no money for study and living. For a student from a poor family (income bracket 1 or 2) this can decide if they study or not. In the bare pipeline this student also cannot complain, because there is no record of why and no person to ask. The unparseable replies in 2.1 are the worst case: the software failed, the applicant was rejected, and nobody knew. The Kazakh personal data law gives the right to an explanation of automated processing and the right to object to it (Law No. 94-V, Art. 19-1(2) and 19-1(3)). In the bare pipeline there is nothing to explain and nobody to object to.

The applicant who is shortlisted by mistake, and the one who loses the place. A wrong shortlist does not hurt the shortlisted person much, because a later stage can still reject them. But the number of places is limited. Another applicant loses the place and never learns about it. The two mistakes are not equal. This is why my harness sends a strong applicant who was rejected to a person (rule strong_record_rejected), but I have no similar rule for weak applicants who were shortlisted.

People the task does not mention.

1. The reviewers. Every escalated case is work for a person. The share is in JUSTIFICATION.md part 6. The review queue does not show the name. But when a reviewer opens the original form to check identity, they see the name, so the bias that I hid from the model can come back at the human step.
2. Applicants that my harness treats differently. An applicant from a university that is not in the fund's list is sent to a person, so it is slower. An applicant with a name of five or more words is refused and goes to manual intake. Part 4b shows this happens to a real name. Applicants under 18 and applicants who say no to automated processing must be handled by hand. If the fund has no staff for this, saying no to automation becomes a punishment.
3. People who never apply because they think a system like this will reject people like them. The notebook cannot count them.
4. The fund. It is legally responsible for the results of the system (Law No. 230-VIII, Art. 8(2)).

If there is an effect that this sample cannot see. If part 1 finds nothing above the floors, a real effect smaller than the McNemar threshold in 1.1 can still exist. It would hurt the same group every year and nobody would see it. Hiding the name and the university closes this way for these two fields, even if the test did not find anything. This is why my blinding does not depend on the result of the test. But it does not remove proxies in the fields that the model still sees: programme, age and income bracket.

What grows over time. If the system is used every year, a small bias adds up in the group of people who get the scholarship. If past winners are later used as examples of good applicants, the model learns the bias again. If the shortlist is used in the next step (an interview, another fund, a public success rate per university), a regional university with fewer shortlisted students looks weaker. Then fewer strong students choose it, and the first bias looks true because of its own results.

## 3. Professional responsibility

What I am responsible for as the developer.

1. Measure before the system runs, and say only what the measurement can show. I did this in Part 1: matched pairs against two floors, plus an order probe and a counterfactual check. I also showed the power of the test. I must not say "no effect" when I only found "no measured effect". I must not say the model is fair because the flips are zero after blinding. They are zero because both twins see the same text (OBSERVATION_3).
2. Make failures visible. The bare pipeline turned unparseable replies into rejections. My harness sends them to a person. Every call, refusal and escalation is in decisions.jsonl. A hash chain shows if someone changes a record later. I tested this in Part 5b.
3. Test the controls on cases where they can be wrong. The refusal path worked in my submitted run on the cases it must stop (Part 4) and on real but unusual applicants (Part 4b). I report the wrong refusal that it made. I do not hide it.
4. Tell the fund the limits and the price: the share of cases for reviewers compared with the plan, the refusal routes that need staff, the problem with long names, the fields the model still sees, and the fact that the score band was set on 60 test applicants and must be set again when anything changes.
5. Say when the system must not run. The notebook checks stop conditions: share of escalations compared with the plan and the limit, share of unparseable replies, and wrong refusals of clean applicants. It prints STOP when a check fails.

What is not done yet (also in JUSTIFICATION.md part 6): a test on real applications, a bigger sample, messages to applicants, a record of the reviewer's final decision, a working objection process, automatic deletion of old records, and a check of my reading of the laws by a lawyer.

What belongs to the fund, not to me. The fund decides the purpose and the criteria. The fund collects consent in the way the law says (Law No. 94-V, Art. 7 and 8). The fund decides the risk level of the system, because the law gives this to the owner (Law No. 230-VIII, Art. 17(1)). The fund tells applicants that AI is used and how to object (Law No. 230-VIII, Art. 7(2) and 21(1); Law No. 94-V, Art. 19-1(2)). The fund gives the review queue to people who can change the model's decision. The fund answers an objection in three working days (Law No. 94-V, Art. 19-1(3)). The fund decides how long records are kept and who can read them.

When the fund and I do not agree. If the fund wants to switch off escalation to save time, I can show what comes back: unparseable replies become silent rejections again, and the borderline decisions that moved with unimportant details are decided by the model alone. I can offer a smaller band. This is a fair choice for the fund, and its price is in JUSTIFICATION.md part 6. But I cannot remove the routes that the law asks for: no automated decision without consent, and every objection must have an answer. If the fund still decides against me, I must write down the decision and what my measurement says about it. I must not sign as if the control is still there. It is the same if the fund wants to show the university to the model again. This is the fund's choice, but they must explain it against my measurement of university against its floor.
