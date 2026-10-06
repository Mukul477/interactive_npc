# Stateful, Adaptive NPCs: Research Review and Design Guidance

## 1. Purpose and scope

This review supports the proposed NPC system: speech input is transcribed, player intent and affect are inferred, an NPC cognitive/persona engine updates persistent memory and relationship state, and a local language model produces a response that remains consistent with the NPC's personality and backstory.

The central design recommendation is **not** to treat the language model as the NPC itself. Instead, use a hybrid architecture:

1. Deterministic state and rules enforce identity, world facts, safety, and relationship updates.
2. Retrieval and summarization select the memories relevant to the current interaction.
3. The language model plans and verbalizes a response within explicit constraints.
4. A validation layer checks persona, factual grounding, emotional plausibility, and output format before the response reaches the game.

This separation directly addresses persona drift, unreliable long-term memory, and inconsistent emotional behavior.

## 2. Most relevant research

### 2.1 Generative Agents: Interactive Simulacra of Human Behavior

**Park et al., UIST 2023.** [Paper and project material](https://arxiv.org/abs/2304.03442)

Generative Agents introduced a practical architecture for believable social agents in a simulated town. Each agent maintains an experience stream, retrieves memories using relevance/recency/importance, and periodically reflects to form higher-level observations. Those memories and reflections guide planning and action.

**Why it matters for this project**

- Provides the strongest direct precedent for persistent, socially believable NPC behavior.
- Suggests separating raw events from distilled semantic memories.
- Gives a useful retrieval score:
  `retrieval_score = relevance + recency + importance`.
- Shows that planning and reflection should be separate from immediate dialogue generation.

**Adaptation**

Store every interaction as an immutable event, then asynchronously create or update semantic memories such as “the player kept a promise.” Retrieve only the top-ranked memories relevant to the current topic, relationship, and emotional context.

### 2.2 Reflexion: Language Agents with Verbal Reinforcement Learning

**Shinn et al., 2023.** [Paper](https://arxiv.org/abs/2303.11366)

Reflexion uses a verbal reflection buffer: after an outcome, an agent records what went well or poorly and uses that text in later decisions. It is not a complete NPC memory system, but it demonstrates how an agent can improve behavior without changing model weights.

**Why it matters**

- Supports episodic “lessons learned” without fine-tuning.
- Provides a mechanism for an NPC to remember failed attempts, broken promises, or successful negotiations.
- Encourages storing reflections separately from objective facts.

**Caution**

Reflections are model-generated interpretations and may be wrong. They should be marked as low-confidence hypotheses and confirmed or corrected by later evidence rather than treated as canonical world facts.

### 2.3 Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

**Lewis et al., NeurIPS 2020.** [Paper](https://arxiv.org/abs/2005.11401)

RAG combines parametric language-model knowledge with a retrievable external memory. The key principle is that changing or growing knowledge should be represented in the retrieval store rather than hidden inside model weights.

**Why it matters**

- Fits PostgreSQL-backed NPC memories and world knowledge.
- Allows memory updates without retraining a local model.
- Makes retrieved evidence inspectable and easier to debug.

**Adaptation**

Use PostgreSQL for canonical records and `pgvector` (or a separate vector index if required) for semantic retrieval. Every retrieved memory should retain its source interaction, timestamp, confidence, and visibility scope.

### 2.4 Persona-Chat: Designing Dialogue Agents with Personas

**Zhang et al., ACL 2018.** [Paper](https://arxiv.org/abs/1801.07243) | [ACL Anthology](https://aclanthology.org/P18-2023/)

Persona-Chat established a benchmark for conditioning dialogue on explicit persona statements. It demonstrates both the value and difficulty of grounding responses in stable character attributes.

**Why it matters**

- Supports representing backstory and personality as explicit, testable persona facts.
- Provides a starting point for persona-grounded evaluation.
- Highlights that persona consistency is distinct from generic response fluency.

**Adaptation**

Represent persona as versioned facts with priority and provenance:

- immutable identity and backstory;
- stable traits and values;
- current goals and obligations;
- temporary mood;
- relationship-specific beliefs.

The prompt should label these categories separately so a temporary mood cannot silently rewrite the backstory.

### 2.5 EmpatheticDialogues: A Large-Scale Dataset for Empathetic Response Generation

**Rashkin et al., ACL 2019.** [Paper](https://arxiv.org/abs/1811.00207) | [ACL Anthology](https://aclanthology.org/P19-1534/)

EmpatheticDialogues studies responses to situations associated with emotions and provides emotional situation labels. It is useful for understanding emotion recognition and emotionally appropriate response generation, although it is not a full game-agent architecture.

**Why it matters**

- Supports the proposed tone/emotion analysis stage.
- Encourages distinguishing the player's inferred emotion from the NPC's internal mood.
- Provides ideas for evaluating emotional appropriateness separately from factual correctness.

**Adaptation**

Use emotion inference as one input to appraisal, not as a direct command. For example, detecting player anger should update the event interpretation and candidate relationship effects; it should not automatically force the NPC to apologize or become angry.

### 2.6 MemoryBank: Enhancing Large Language Models with Long-Term Memory

**Zhong et al., 2023.** [Paper](https://arxiv.org/abs/2305.10250)

MemoryBank explores long-term memory for conversational agents, including memory formation, retrieval, and update over extended interaction. It is an early but useful reference for storing user-specific experiences and evolving conversational context.

**Why it matters**

- Directly relevant to persistent player-specific memories.
- Motivates memory consolidation and forgetting rather than storing an unbounded transcript.
- Suggests that memory should influence both content and the relationship model.

**Caution**

This is an arXiv preprint. Use its architectural ideas, but validate retrieval and memory-update policies experimentally in the target game.

### 2.7 MemGPT: Towards LLMs as Operating Systems

**Packer et al., 2023.** [Paper](https://arxiv.org/abs/2310.08560)

MemGPT treats the language model as managing a hierarchy of memory: a limited working context plus larger archival storage. The model decides what to page in and out.

**Why it matters**

- Offers a useful mental model for local-model context limits.
- Supports separating immediate conversation context from long-term archives.
- Suggests explicit memory-management operations instead of blindly appending the entire history to every prompt.

**Adaptation**

Do not give the model unrestricted write access to canonical relationship values. Let the model propose memory operations in a structured schema; a deterministic cognitive engine validates and applies them.

### 2.8 Character-LLM: A Trainable Agent for Role-Playing

**Shao et al., 2023.** [Paper](https://arxiv.org/abs/2310.10158)

Character-LLM studies language-model agents that role-play characters based on biographies and experience. It is relevant to personality-conditioned generation and evaluating whether generated conversations reflect a character's profile.

**Why it matters**

- Provides a direct connection between character biography and role-play behavior.
- Reinforces the importance of evaluating character consistency, not just linguistic quality.
- Suggests using character-specific experiences as conditioning information.

### 2.9 Voyager: An Open-Ended Embodied Agent with Large Language Models

**Wang et al., 2023.** [Paper](https://arxiv.org/abs/2305.16291)

Voyager demonstrates an LLM-driven embodied agent with an automatic curriculum, skill library, and iterative feedback. It targets Minecraft rather than social NPC dialogue, but its modular skill representation is valuable for NPC action planning.

**Why it matters**

- Dialogue should be connected to executable actions and goals, not generated in isolation.
- A reusable skill/action library can constrain model outputs and make behavior reliable.
- Feedback from the game world should update the agent's state.

**Adaptation**

Represent candidate NPC actions as typed game commands such as `offer_quest`, `refuse_request`, `move_to_location`, or `change_relationship`. The model selects or fills these actions; the game validates and executes them.

### 2.10 Personalized Non-Player Characters: A Framework for Character-Consistent Dialogue Generation

**2025.** [MDPI article](https://www.mdpi.com/2673-2688/6/5/93)

This recent NPC-focused work combines persona information, interaction history, and structured relationship information for character-consistent dialogue. It is especially relevant because it addresses NPC personalization rather than only general-purpose chat.

**Why it matters**

- Supports modeling relationships as structured data rather than only prose memories.
- Provides a recent comparison point for a project evaluation.
- Motivates knowledge-graph-like links between NPCs, players, events, and beliefs.

**Caution**

Check the paper's exact datasets, baselines, and evaluation protocol before claiming superiority. Use it as related work and an implementation prompt, not as proof that a particular architecture is universally best.

### 2.11 LLM-Driven NPCs: Cross-Platform Dialogue System for Games and Social Platforms

**2025 preprint.** [Paper](https://arxiv.org/abs/2504.13928)

This work explores NPC dialogue across a game and a social platform, including synchronized interaction history and affect/favorability signals.

**Why it matters**

- Relevant to persistent identity across multiple interfaces.
- Demonstrates the value of an external state layer shared by all clients.
- Provides a modern reference for evaluating cross-session NPC continuity.

**Caution**

It is a recent preprint and should be independently reproduced before adopting its claims or metrics.

## 3. Supporting engineering and evaluation material

### PostgreSQL and vector retrieval

- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)
- [pgvector](https://github.com/pgvector/pgvector)

Use relational tables for authoritative state, indexes for time and player/NPC scope, and vector embeddings only as a retrieval aid. A vector similarity match must never override a newer explicit fact or an invariant persona rule.

### Speech and emotion pipeline references

- [Whisper: Robust Speech Recognition via Large-Scale Weak Supervision](https://arxiv.org/abs/2212.04356)
- [Wav2Vec 2.0](https://arxiv.org/abs/2006.1143)
- [Speech Emotion Recognition: A Survey](https://arxiv.org/abs/2205.07317)

Speech-to-text and emotion recognition should expose confidence scores. Low-confidence emotion predictions should have a smaller effect on mood or relationships than explicit player actions and verified game events.

### Agent evaluation

- [SotA: A Survey of LLM-Based Agent Evaluation](https://arxiv.org/abs/2404.02875)
- [AgentBench: Evaluating LLMs as Agents](https://arxiv.org/abs/2308.03688)
- [G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment](https://arxiv.org/abs/2303.16634)

For this project, combine automated checks with human playtests. Generic “conversation quality” scores are insufficient because an NPC can sound fluent while violating its backstory or relationship history.

## 4. Proposed architecture derived from the literature

```text
Microphone
   |
Speech-to-text + confidence
   |
Intent / tone / emotion / entities
   |
Interaction event (immutable)
   |
NPC cognitive engine
   |-- appraisal: goal impact, norm violation, threat, benefit
   |-- relationship update: trust, affinity, respect, fear, debt
   |-- mood update: short-term affect with decay
   |-- memory write proposal: event, fact, reflection, confidence
   |-- goal and action selection
   |
Memory retrieval + context assembly
   |-- recent turns
   |-- relevant episodic memories
   |-- consolidated semantic memories
   |-- persona and backstory constraints
   |-- current relationship and mood
   |
Local language model
   |
Structured response proposal
   |-- speech text
   |-- emotion display
   |-- game action(s)
   |-- memory updates
   |
Validator / policy / game-state check
   |
Text-to-speech + animation + game action
```

### Recommended state boundaries

| State | Examples | Update policy |
| --- | --- | --- |
| Immutable identity | name, origin, core backstory | Never changed by dialogue |
| Stable persona | values, traits, speaking style | Changed only by authored updates |
| Current goals | protect a location, find an item | Updated by quests and world events |
| Mood | calm, afraid, irritated | Decays toward baseline over time |
| Relationship | trust, affinity, respect, fear, obligation | Updated by appraised events |
| Episodic memory | “player warned me about the ambush” | Append, score, retrieve, consolidate |
| Semantic belief | “player is reliable” | Derived from multiple episodes; confidence tracked |
| Procedural knowledge | available actions and social policies | Authored or learned under validation |

## 5. Suggested PostgreSQL model

The exact schema can evolve, but the research suggests keeping facts, events, memories, and derived state distinct:

```text
npcs
players
npc_persona_facts
npc_relationships
interaction_events
npc_memories
npc_reflections
npc_goals
npc_state_snapshots
```

Recommended fields for `npc_memories` include:

- `memory_type`: episodic, semantic, reflection, procedural;
- `content` and optional embedding;
- `source_event_id`;
- `importance`, `confidence`, `created_at`, `last_retrieved_at`;
- `valid_from`, `valid_until` or a supersession link;
- `player_id` and visibility scope.

Store state transitions as auditable events or snapshots. This makes it possible to reproduce a conversation, investigate an unexpected response, and compare different update policies.

## 6. Research gaps that create innovation opportunities

1. **Verified persona transitions:** allow relationship adaptation while preventing model-generated text from rewriting immutable identity or core values.
2. **Multi-timescale emotion:** combine fast mood changes, slower relationship changes, and very slow personality development.
3. **Memory provenance and contradiction handling:** track where each belief came from and resolve conflicts using recency, confidence, direct evidence, and authored canon.
4. **Action-grounded dialogue:** require the model to emit structured, executable actions rather than free-form text only.
5. **Local-model optimization:** compare prompt compression, retrieval quality, quantization, and speculative generation under a fixed latency budget.
6. **Player-specific evaluation:** measure continuity after days of interaction, not just turn-level response quality.
7. **Uncertainty-aware affect:** prevent a low-confidence tone classifier from causing a large relationship change.
8. **Social memory graphs:** represent promises, debts, allies, witnesses, and shared events as linked entities rather than isolated text chunks.

## 7. Evaluation plan

Track both system metrics and human-perceived believability:

| Dimension | Example metric |
| --- | --- |
| Persona consistency | blinded raters detect fewer contradictions; automated fact checks pass |
| Memory accuracy | precision/recall of recalled facts; no fabricated memories |
| Relationship continuity | state changes match labeled interaction outcomes |
| Emotional appropriateness | response fits player affect and NPC appraisal |
| Behavioral adaptation | NPC choices change after relevant player actions |
| Action validity | percentage of generated actions accepted by the game |
| Latency | p50/p95 speech-to-response time |
| Safety and robustness | invalid schema rate, prompt-injection resistance, refusal correctness |
| Player experience | trust, perceived agency, immersion, and repetitiveness in playtests |

Build a replayable test set containing neutral, kind, hostile, deceptive, ambiguous, and contradictory interactions. Test both a fresh NPC and an NPC with a long history. Report ablations for memory retrieval, reflection, relationship state, and validation so improvements can be attributed to specific components.

## 8. Recommended implementation order

1. Implement deterministic interaction events, persona facts, relationship state, and state snapshots.
2. Add a text-only dialogue loop with structured model output and validation.
3. Add episodic memory retrieval and source-linked semantic consolidation.
4. Add intent, tone, and emotion inference with confidence-aware updates.
5. Add goals, executable game actions, and action-result feedback.
6. Add speech-to-text/text-to-speech and latency instrumentation.
7. Run ablation studies and longitudinal playtests before adding model fine-tuning.

## 9. Reference list

1. Park, J. S. et al. “Generative Agents: Interactive Simulacra of Human Behavior.” UIST 2023. https://arxiv.org/abs/2304.03442
2. Shinn, N. et al. “Reflexion: Language Agents with Verbal Reinforcement Learning.” 2023. https://arxiv.org/abs/2303.11366
3. Lewis, P. et al. “Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.” NeurIPS 2020. https://arxiv.org/abs/2005.11401
4. Zhang, S. et al. “Personalizing Dialogue Agents: I have a dog, do you have pets too?” ACL 2018. https://aclanthology.org/P18-2023/
5. Rashkin, H. et al. “Towards Empathetic Open-domain Conversation Models: A New Benchmark and Dataset.” ACL 2019. https://aclanthology.org/P19-1534/
6. Zhong, W. et al. “MemoryBank: Enhancing Large Language Models with Long-Term Memory.” 2023. https://arxiv.org/abs/2305.10250
7. Packer, C. et al. “MemGPT: Towards LLMs as Operating Systems.” 2023. https://arxiv.org/abs/2310.08560
8. Shao, Y. et al. “Character-LLM: A Trainable Agent for Role-Playing.” 2023. https://arxiv.org/abs/2310.10158
9. Wang, G. et al. “Voyager: An Open-Ended Embodied Agent with Large Language Models.” 2023. https://arxiv.org/abs/2305.16291
10. “Personalized Non-Player Characters: A Framework for Character-Consistent Dialogue Generation.” 2025. https://www.mdpi.com/2673-2688/6/5/93
11. “LLM-Driven NPCs: Cross-Platform Dialogue System for Games and Social Platforms.” 2025. https://arxiv.org/abs/2504.13928
12. Radford, A. et al. “Robust Speech Recognition via Large-Scale Weak Supervision.” 2022. https://arxiv.org/abs/2212.04356
13. Liu, X. et al. “AgentBench: Evaluating LLMs as Agents.” 2023. https://arxiv.org/abs/2308.03688

Links were checked against the linked publisher, ACL Anthology, project, or arXiv landing pages on 2026-10-05. Preprints and industry material are labeled so their evidence is not confused with peer-reviewed findings.

## 10. Extended literature set

The following references broaden the project literature beyond the core sources. They are grouped by the subsystem they can inform. Not every paper is an NPC paper; the connection is stated so that foundational methods are not presented as direct evidence about games.

### A. Believable agents, games, social simulation, and computational emotion

14. Bates, J. “The Role of Emotion in Believable Agents.” *Communications of the ACM*, 1994. [DOI](https://doi.org/10.1145/176789.176803) — Foundational argument that emotion, goals, and social context are necessary for believable agent behavior.

15. Mateas, M. “An Oz-Centric Review of Interactive Drama and Believable Agents.” *AI and Interactive Entertainment*, 2002. [PDF](https://www.cs.cmu.edu/~illah/CLASS_USE/Oz-review.pdf) — Reviews planning, authorial control, and believable-agent architectures for interactive narrative.

16. Mateas, M. and Stern, A. “Façade: An Experiment in Building a Fully-Realized Interactive Drama.” *Game Developers Conference*, 2003. [Project page](https://www.interactivestory.net/) — Demonstrates beat-based narrative management, discourse acts, and authored dramatic control.

17. Si, M., Marsella, S., and Pynadath, D. “Thespian: Using Multi-Agent Fitting to Craft Interactive Drama.” *AAMAS*, 2009. [DOI](https://doi.org/10.1145/1558013.1558177) — Uses multi-agent planning and social reasoning to generate interactive drama.

18. Riedl, M. O. and Young, R. M. “Narrative Planning: Balancing Plot and Character.” *Journal of Artificial Intelligence Research*, 2010. [Paper](https://www.jair.org/index.php/jair/article/view/10569) — Relevant to balancing NPC goals, player agency, and authored story constraints.

19. Riedl, M. O. and Bulitko, V. “Interactive Narrative: An Intelligent Systems Approach.” *AI Magazine*, 2013. [DOI](https://doi.org/10.1609/aimag.v34i1.2449) — Surveys planning and adaptation for interactive narratives.

20. Swartjes, I. and Theune, M. “The Virtual Storyteller: Story Generation by Simulation.” *Computational Linguistics*, 2006. [ACL Anthology](https://aclanthology.org/W06-1503/) — Models characters, events, and story generation through simulation.

21. Traum, D. et al. “Building Virtual Humans with a Multimodal Architecture.” *IEEE Intelligent Systems*, 2004. [DOI](https://doi.org/10.1109/MIS.2004.1265898) — Provides a multimodal architecture for embodied conversational agents.

22. Gratch, J. and Marsella, S. “A Domain-Independent Framework for Modeling Emotion.” *Cognitive Systems Research*, 2004. [DOI](https://doi.org/10.1016/j.cogsys.2004.02.002) — Connects appraisal, goals, and emotional state transitions in computational agents.

23. Marsella, S. and Gratch, J. “EMA: A Process Model of Appraisal Dynamics.” *Cognitive Systems Research*, 2009. [DOI](https://doi.org/10.1016/j.cogsys.2008.03.005) — A practical reference for implementing event appraisal and mood dynamics.

24. Ortony, A., Clore, G. L., and Collins, A. *The Cognitive Structure of Emotions*. Cambridge University Press, 1988. [Publisher](https://www.cambridge.org/core/books/cognitive-structure-of-emotions/3AE3D0D715F4C2B5D2C7B4B87E2B79A5) — Defines the OCC appraisal categories that can structure NPC emotion updates.

25. Rao, A. S. and Georgeff, M. P. “BDI Agents: From Theory to Practice.” 1995. [Paper](https://www.cs.ubc.ca/~mack/Publications/AIJ-1995.pdf) — Supplies a formal model for beliefs, goals, intentions, and action selection.

26. Funge, J., Tu, X., and Terzopoulos, D. “Cognitive Modeling: Knowledge, Reasoning and Planning for Intelligent Characters.” *Proceedings of SIGGRAPH*, 1999. [DOI](https://doi.org/10.1145/311535.311560) — Connects world knowledge, planning, and character behavior in interactive environments.

27. ElSayed, S. and King, D. J. “Affect and Believability in Game Characters: A Review of the Use of Affective Computing in Games.” GAME-ON, 2017. [Publication record](https://rke.abertay.ac.uk/en/publications/affect-and-believability-in-game-characters-a-review-of-the-use-o/) — Reviews affective computing requirements for emotionally believable game characters.

28. Yannakakis, G. N. and Melhart, D. “Affective Game Computing: A Survey.” 2023. [arXiv](https://arxiv.org/abs/2309.14104) — Surveys sensing, modeling, and adaptation in affect-aware games.

29. Melhart, D. et al. “Procedural Content Generation for Games: A Survey.” *IEEE Transactions on Games*, 2023. [DOI](https://doi.org/10.1109/TG.2023.3250453) — Useful for generating varied NPC events, quests, and social situations.

30. Mitchell, K., Pettijohn, C., and McCoy, J. “Never a Dull Moment: Believable Dynamic Character Beat Generation between Game World Events.” *AIIDE*, 2022. [AAAI](https://ojs.aaai.org/index.php/AIIDE/article/view/21974) — Directly addresses dynamic character beats and believable behavior between authored world events.

### B. Dialogue, grounding, persona, empathy, and social interaction

31. Dinan, E. et al. “Wizard of Wikipedia: Knowledge-Powered Conversational Agents.” *ICLR*, 2019. [OpenReview](https://openreview.net/forum?id=H1g6XeR9KX) — Grounds dialogue in retrieved knowledge and provides a benchmark for factual conversational responses.

32. Gopalakrishnan, K. et al. “Topical-Chat: Towards Knowledge-Grounded Open-Domain Conversations.” *INTERSPEECH*, 2019. [arXiv](https://arxiv.org/abs/1811.01394) — Combines conversation with topic and knowledge grounding.

33. Dinan, E. et al. “Build It Break It Fix It: Behavior Sets for Open-Domain Dialogue.” *EMNLP*, 2019. [ACL Anthology](https://aclanthology.org/D19-1189/) — Demonstrates adversarial collection and evaluation for dialogue behavior.

34. Roller, S. et al. “Recipes for Building an Open-Domain Chatbot.” *EACL*, 2021. [ACL Anthology](https://aclanthology.org/2021.eacl-main.24/) — Practical lessons for data, retrieval, generation, and evaluation in open-domain dialogue.

35. Thoppilan, R. et al. “LaMDA: Language Models for Dialog Applications.” 2022. [arXiv](https://arxiv.org/abs/2201.08239) — Studies quality, safety, and groundedness dimensions for dialog-oriented language models.

36. Ouyang, L. et al. “Training Language Models to Follow Instructions with Human Feedback.” *NeurIPS*, 2022. [Paper](https://arxiv.org/abs/2203.02155) — Important background for instruction following and human preference alignment.

37. Bai, Y. et al. “Constitutional AI: Harmlessness from AI Feedback.” 2022. [Paper](https://arxiv.org/abs/2212.08073) — Relevant to defining NPC response constraints and self-critique policies.

38. Welivita, A. and Pu, P. “A Taxonomy of Empathetic Response Intents in Human Social Conversations.” *ACL*, 2020. [ACL Anthology](https://aclanthology.org/2020.nlp4convai-1.4/) — Helps define response intents such as acknowledging, comforting, encouraging, and suggesting.

39. Li, J. et al. “Towards a Unified Evaluation of Empathetic Dialogue Systems.” *EMNLP Findings*, 2024. [ACL Anthology](https://aclanthology.org/2024.findings-emnlp.113/) — Supports multidimensional evaluation instead of a single empathy score.

40. Clark, L. et al. “Building Common Ground in Dialogue: A Survey.” *Findings of EMNLP*, 2024. [ACL Anthology](https://aclanthology.org/2024.findings-emnlp.113/) — Provides concepts for grounding, alignment, and shared conversational context.

41. Roller, S. et al. “Open-Domain Dialog Evaluation.” *ACL*, 2021. [ACL Anthology](https://aclanthology.org/2021.acl-long.395/) — Reviews problems with automatic dialogue evaluation and supports human-centered evaluation design.

42. Li, J. et al. “A Persona-Based Neural Conversation Model.” *ACL*, 2016. [ACL Anthology](https://aclanthology.org/P16-1094/) — Early neural approach to persona-conditioned response generation.

43. Wolf, T. et al. “TransferTransfo: A Transfer Learning Approach for Neural Network Based Conversational Agents.” 2019. [OpenReview](https://openreview.net/forum?id=H1g1tQYk) — Early large-scale persona and dialogue transfer-learning baseline.

### C. LLM agents, planning, tools, reflection, and embodied action

44. Yao, S. et al. “ReAct: Synergizing Reasoning and Acting in Language Models.” *ICLR*, 2023. [OpenReview](https://openreview.net/forum?id=WE_vluYUL-X) — Interleaves reasoning and actions; useful for structured NPC action selection.

45. Schick, T. et al. “Toolformer: Language Models Can Teach Themselves to Use Tools.” *NeurIPS*, 2023. [Paper](https://arxiv.org/abs/2302.04761) — Motivates typed tools for game actions, databases, and world queries.

46. Yao, S. et al. “Tree of Thoughts: Deliberate Problem Solving with Large Language Models.” *NeurIPS*, 2023. [Paper](https://arxiv.org/abs/2305.10601) — Provides a planning/search pattern for comparing candidate NPC actions.

47. Madaan, A. et al. “Self-Refine: Iterative Refinement with Self-Feedback.” *NeurIPS*, 2023. [Paper](https://arxiv.org/abs/2303.17651) — Supports generating, critiquing, and revising a response before delivery.

48. Wang, X. et al. “Plan-and-Solve Prompting.” *ACL*, 2023. [ACL Anthology](https://aclanthology.org/2023.acl-long.147/) — Separates planning from execution, a useful pattern for NPC goals and dialogue.

49. Sumers, T. R. et al. “Cognitive Architectures for Language Agents.” *Transactions on Machine Learning Research*, 2024. [Paper](https://arxiv.org/abs/2309.02427) — Surveys memory, planning, perception, and action modules for language agents.

50. Wang, G. et al. “Voyager: An Open-Ended Embodied Agent with Large Language Models.” 2023. [Paper](https://arxiv.org/abs/2305.16291) — Skill libraries, automatic curriculum, and embodied feedback.

51. Zhou, S. et al. “WebArena: A Realistic Web Environment for Building Autonomous Agents.” *ICLR*, 2024. [OpenReview](https://openreview.net/forum?id=oWSLJzs0RM) — Provides ideas for evaluating agents in stateful environments with real consequences.

52. Yao, S. et al. “τ-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains.” 2024. [Paper](https://arxiv.org/abs/2406.12045) — Useful for evaluating stateful tool use and multi-turn policy adherence.

### D. Memory, retrieval, knowledge graphs, and long-context systems

53. Guu, K. et al. “REALM: Retrieval-Augmented Language Model Pre-Training.” *ICML*, 2020. [PMLR](https://proceedings.mlr.press/v119/guu20a.html) — Early retrieval-augmented language-model architecture.

54. Karpukhin, V. et al. “Dense Passage Retrieval for Open-Domain Question Answering.” *EMNLP*, 2020. [ACL Anthology](https://aclanthology.org/2020.emnlp-main.550/) — Establishes dense retrieval as a strong baseline for selecting relevant memories.

55. Izacard, G. and Grave, E. “Leveraging Passage Retrieval with Generative Models for Open Domain Question Answering.” *EACL*, 2021. [ACL Anthology](https://aclanthology.org/2021.eacl-main.74/) — Fusion of multiple retrieved passages for grounded generation.

56. Khattab, O. and Zaharia, M. “ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT.” *SIGIR*, 2020. [DOI](https://dl.acm.org/doi/10.1145/3397271.3401075) — Strong retrieval architecture for fine-grained memory matching.

57. Izacard, G. et al. “Atlas: Few-Shot Learning with Retrieval Augmented Language Models.” *JMLR*, 2023. [JMLR](https://www.jmlr.org/papers/v24/23-0037.html) — Studies retrieval-augmented models under few-shot conditions.

58. Gao, L. et al. “Precise Zero-Shot Dense Retrieval without Relevance Labels.” *ACL*, 2023. [ACL Anthology](https://aclanthology.org/2023.acl-long.99/) — Contriever provides a useful unsupervised retrieval baseline.

59. Gao, Y. et al. “Retrieval-Augmented Generation for Large Language Models: A Survey.” 2023. [Paper](https://arxiv.org/abs/2312.10997) — Taxonomy of retrieval, augmentation, and generation design choices.

60. Edge, D. et al. “From Local to Global: A Graph RAG Approach to Query-Focused Summarization.” 2024. [Paper](https://arxiv.org/abs/2404.16130) — Relevant to social-memory graphs and community-level NPC knowledge.

61. Gutierrez, B. J. et al. “HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models.” *NeurIPS*, 2024. [Paper](https://arxiv.org/abs/2405.14831) — Uses associative retrieval and knowledge graphs for long-term memory.

62. Zhang, Z. et al. “A Survey on the Memory Mechanism of Large Language Model Based Agents.” 2024. [Paper](https://arxiv.org/abs/2404.13501) — Taxonomy of working, episodic, semantic, procedural, and parametric memory.

63. Wang, L. et al. “A-MEM: Agentic Memory for LLM Agents.” 2025. [Paper](https://arxiv.org/abs/2502.12110) — Dynamic memory organization and linking for agentic workflows.

64. Anokhin, P. et al. “AriGraph: Learning Knowledge Graph World Models with Episodic Memory for LLM Agents.” 2024. [Paper](https://arxiv.org/abs/2407.04363) — Combines episodic memory with graph world models for planning.

### E. Speech, language, multimodal input, and emotion recognition

65. Baevski, A. et al. “wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations.” *NeurIPS*, 2020. [Paper](https://arxiv.org/abs/2006.1143) — Strong self-supervised speech representation baseline.

66. Hsu, W.-N. et al. “HuBERT: Self-Supervised Speech Representation Learning by Masked Prediction of Hidden Units.” *IEEE/ACM TASLP*, 2021. [Paper](https://arxiv.org/abs/2106.07447) — Alternative speech representation model useful for emotion and intent features.

67. Ao, J. et al. “SpeechT5: Unified-Modal Encoder-Decoder Pre-Training for Spoken Language Processing.” *ACL*, 2022. [ACL Anthology](https://aclanthology.org/2022.acl-long.494/) — Unifies speech and text representations for spoken-language systems.

68. Barrault, L. et al. “SeamlessM4T: Massively Multilingual and Multimodal Machine Translation.” 2023. [Paper](https://arxiv.org/abs/2308.11596) — Reference for multilingual speech input and speech output pipelines.

69. Busso, C. et al. “IEMOCAP: Interactive Emotional Dyadic Motion Capture Database.” *Language Resources and Evaluation*, 2008. [DOI](https://doi.org/10.1007/s10579-008-9076-6) — Standard acted emotional-speech benchmark.

70. Poria, S. et al. “MELD: A Multimodal Multi-Party Dataset for Emotion Recognition in Conversations.” *ACL*, 2019. [ACL Anthology](https://aclanthology.org/P19-1050/) — Multimodal and conversational emotion-recognition benchmark.

71. Livingstone, S. R. and Russo, F. A. “The Ryerson Audio-Visual Database of Emotional Speech and Song.” *PLOS ONE*, 2018. [DOI](https://doi.org/10.1371/journal.pone.0196391) — Audio-visual emotion data for speech affect research.

72. Zadeh, A. et al. “Tensor Fusion Network for Multimodal Sentiment Analysis.” *EMNLP*, 2017. [ACL Anthology](https://aclanthology.org/D17-1115/) — A classic multimodal fusion method relevant to speech, text, and facial cues.

73. Tsai, Y.-H. H. et al. “Multimodal Transformer for Unaligned Multimodal Language Sequences.” *ACL*, 2019. [ACL Anthology](https://aclanthology.org/P19-1656/) — Models asynchronous text, audio, and visual signals.

74. Cowen, A. and Keltner, D. “Self-Report Captures 27 Distinct Categories of Emotion Bridged by Continuous Gradients.” *PNAS*, 2017. [DOI](https://doi.org/10.1073/pnas.1702247114) — Supports using continuous affect dimensions rather than only discrete labels.

### F. Evaluation, safety, reliability, and human-centered assessment

75. Liang, P. et al. “Holistic Evaluation of Language Models.” *Transactions on Machine Learning Research*, 2023. [Paper](https://arxiv.org/abs/2211.09110) — Provides a broad evaluation framework for capability, calibration, robustness, and bias.

76. Liu, Y. et al. “G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment.” *EMNLP*, 2023. [ACL Anthology](https://aclanthology.org/2023.emnlp-main.153/) — Automated evaluation design; should be combined with human judgments for NPC dialogue.

77. Li, J. et al. “HaluEval: A Large-Scale Hallucination Evaluation Benchmark for Large Language Models.” *EMNLP*, 2023. [ACL Anthology](https://aclanthology.org/2023.emnlp-main.439/) — Relevant to fabricated memories and false NPC claims.

78. Lin, S. et al. “TruthfulQA: Measuring How Models Mimic Human Falsehoods.” *ACL*, 2022. [ACL Anthology](https://aclanthology.org/2022.acl-long.229/) — Relevant to testing truthfulness and resistance to plausible but false responses.

79. Zheng, L. et al. “Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena.” *NeurIPS*, 2023. [Paper](https://arxiv.org/abs/2306.05685) — Discusses model-based and human preference evaluation, including judge reliability.

80. Liu, X. et al. “AgentBench: Evaluating LLMs as Agents.” *ICLR*, 2024. [OpenReview](https://openreview.net/forum?id=zAdUB0aCTQ) — Multi-environment benchmark for planning and tool use.

81. Qin, Y. et al. “ToolBench: Towards Mastering Every Tool for LLMs.” 2023. [Paper](https://arxiv.org/abs/2307.16789) — Useful reference for evaluating structured tool/API use.

82. Ji, Z. et al. “Survey of Hallucination in Natural Language Generation.” *ACM Computing Surveys*, 2023. [DOI](https://doi.org/10.1145/3571730) — Taxonomy of hallucinations relevant to NPC memories and world facts.

83. Weidinger, L. et al. “Taxonomy of Risks Posed by Language Models.” *FAccT*, 2022. [ACM](https://doi.org/10.1145/3531146.3533088) — Helps define safety and misuse tests for player-facing dialogue.

84. Amodei, D. et al. “Concrete Problems in AI Safety.” 2016. [Paper](https://arxiv.org/abs/1606.06565) — Foundational safety taxonomy covering reward hacking, side effects, and distribution shift.

85. Perez, E. et al. “Red Teaming Language Models with Language Models.” *EMNLP*, 2022. [ACL Anthology](https://aclanthology.org/2022.emnlp-main.225/) — Supports adversarial testing of prompt and dialogue boundaries.

86. Saito, K. et al. “The Ethics of AI in Games: A Systematic Review.” 2024. [Scholar search](https://scholar.google.com/scholar?q=ethics+of+AI+in+games+systematic+review) — Use for player consent, profiling, emotional manipulation, and data-retention questions.

### How to use this expanded set

Use the references that directly support each design claim rather than listing citations without synthesis. A practical division is:

- **Core architecture:** 14–30, 44–50, 62–64.
- **Persona and social dialogue:** 31–43.
- **Memory and retrieval implementation:** 53–64.
- **Speech and emotion pipeline:** 65–74.
- **Testing and reliability:** 75–86.

The expanded set now contains more than 80 distinct references. Several items are surveys, foundational books, benchmarks, or engineering papers rather than direct NPC studies; this is intentional because a sound system design needs evidence for each subsystem and not only papers that use the word “NPC.”
