# Literature Analysis and Research Gaps

## 1. Purpose

This document analyzes the literature collected in `NPC_RESEARCH_REVIEW.md` and converts it into a research position for a stateful, adaptive NPC system. The objective is not to claim that one existing method solves the problem. The objective is to identify which ideas are mature, where the literature remains incomplete, and what contribution this project can make.

The proposed system combines:

- speech-to-text and player intent/affect recognition;
- persistent episodic and semantic memory;
- explicit persona and backstory constraints;
- dynamic mood and player-specific relationship state;
- local language-model dialogue generation;
- structured game actions and validation.

## 2. Problem definition

Traditional NPCs commonly separate dialogue, behavior, and state into authored scripts. This gives designers control, but it limits adaptation and creates repetitive interactions. LLM-based NPCs improve linguistic variety, but a language model alone does not guarantee:

1. accurate long-term memory;
2. consistent persona and backstory;
3. emotionally plausible state transitions;
4. stable relationships across sessions;
5. valid interaction with the game world;
6. reproducible and measurable behavior.

The research problem can therefore be stated as:

> How can a game NPC maintain an auditable internal state and use it to produce persona-consistent, emotionally appropriate, relationship-aware dialogue and actions over long-term interaction with a player?

## 3. Synthesis of the literature

### 3.1 Believable-agent and game-AI research

Classical believable-agent research established that convincing characters require more than text generation. Bates, Mateas, Stern, Gratch, Marsella, Riedl, and related work connect character believability to goals, emotions, social context, planning, authorial control, and narrative consequences.

The key contribution of this tradition is **behavioral structure**. A character should have beliefs, desires, intentions, goals, and emotional reactions that influence actions. Narrative systems also show that fully autonomous behavior can conflict with pacing and authored story requirements, so game agents need controllable boundaries.

**Strength for this project:** strong conceptual foundations for goals, appraisal, planning, and authorial control.

**Limitation:** most systems predate modern language models and do not solve open-ended speech, semantic memory retrieval, or natural-language persona consistency at scale.

### 3.2 Generative and language-model agents

Generative Agents, ReAct, Reflexion, Voyager, MemGPT, and related work show that language models can plan, retrieve information, reflect on experience, and call tools. These systems move beyond a single prompt-response loop by adding memory, planning, and action modules.

Their common architectural lesson is that an LLM should be treated as one component of an agent, not as the complete source of truth. External memory, tools, environment feedback, and structured actions are necessary for persistent behavior.

**Strength for this project:** provides practical patterns for memory retrieval, reflection, planning, tool use, and action execution.

**Limitation:** many evaluations focus on task completion or benchmark scores rather than long-term social continuity with one player. Their memory policies also frequently rely on model-generated summaries whose factual status and provenance are unclear.

### 3.3 Persona and dialogue research

Persona-Chat, Character-LLM, persona-aware role-playing work, Wizard of Wikipedia, Topical-Chat, and empathy research show that dialogue quality improves when responses are grounded in persona, context, knowledge, and social intent.

This literature distinguishes several concerns that are often incorrectly combined:

- **persona consistency:** whether the character behaves according to stable traits and biography;
- **factual grounding:** whether claims are supported by world knowledge or retrieved evidence;
- **social appropriateness:** whether the response fits the interaction;
- **empathy:** whether the response recognizes and responds appropriately to another person's affect;
- **fluency:** whether the language is natural.

These dimensions are related but not interchangeable. A fluent response can violate the persona, and an empathetic response can still contain a false memory.

**Strength for this project:** supplies datasets, evaluation concepts, and grounding methods for persona-aware dialogue.

**Limitation:** most persona benchmarks use short conversations and fixed persona statements. They do not fully model mutable relationships, mood decay, conflicting memories, or game-world actions.

### 3.4 Memory and retrieval research

RAG, REALM, DPR, ColBERT, MemGPT, MemoryBank, HippoRAG, Generative Agents, and memory surveys collectively support a layered memory design:

- working memory for the current turn;
- episodic memory for events and experiences;
- semantic memory for consolidated beliefs and facts;
- procedural memory for skills and policies;
- authoritative world state outside the language model.

The literature strongly supports retrieval rather than placing the entire history in the prompt. It also supports hybrid retrieval using semantic similarity, recency, importance, metadata, and graph relationships.

**Strength for this project:** mature design patterns exist for external memory and retrieval.

**Limitation:** retrieval success is not the same as memory correctness. A system can retrieve a relevant but outdated, ambiguous, or false memory. Long-term memory formation, contradiction resolution, forgetting, and provenance remain under-specified for game NPCs.

### 3.5 Emotion, appraisal, and affective computing

OCC theory, EMA, affective game computing, EmpatheticDialogues, IEMOCAP, MELD, and multimodal speech-emotion work provide complementary perspectives:

- appraisal models explain why an event matters to a specific agent;
- affective computing detects or models player and agent emotion;
- empathetic dialogue work evaluates emotionally appropriate language;
- speech and multimodal models provide input signals.

The most important design principle is that emotion should not be a direct label-to-response mapping. Player tone is an uncertain observation. The NPC should appraise the event according to its goals, values, beliefs, and relationship with the player before updating mood or behavior.

**Strength for this project:** provides theories and datasets for emotion recognition, appraisal, and affective response.

**Limitation:** emotion-recognition benchmarks often use acted or curated data and discrete labels. They do not establish how uncertain affect predictions should change a persistent NPC relationship over many sessions.

### 3.6 Evaluation, reliability, and safety

AgentBench, MT-Bench, G-Eval, HaluEval, TruthfulQA, hallucination surveys, red-teaming work, and agent-evaluation surveys demonstrate that fluent outputs are not sufficient evidence of reliable behavior.

For this project, evaluation must cover at least five levels:

1. **Input:** speech transcription, intent, and affect confidence.
2. **State:** memory writes and relationship/mood transitions.
3. **Generation:** persona, grounding, and emotional appropriateness.
4. **Action:** validity and consistency with game rules.
5. **Experience:** player-perceived continuity, agency, and believability.

**Strength for this project:** strong general evaluation methods exist.

**Limitation:** no single standard benchmark captures long-term NPC identity, memory, relationships, emotion, dialogue, and game action together.

## 4. Comparative analysis

| Research direction | What it solves well | What it does not solve for this project |
| --- | --- | --- |
| Scripted behavior trees | Predictability, timing, and game integration | Open-ended language and long-term social adaptation |
| BDI and appraisal agents | Interpretable goals, beliefs, intentions, and emotion | Natural language breadth and scalable memory retrieval |
| Persona-conditioned dialogue | Character style and biography grounding | Persistent relationships, world-state validity, and action execution |
| RAG and vector memory | External knowledge and scalable retrieval | Truth maintenance, contradiction resolution, and emotional meaning |
| Generative-agent architectures | Reflection, planning, and social simulation | Reliable provenance and game-specific evaluation |
| LLM tool-use agents | Structured actions and environment interaction | Stable identity and long-term interpersonal continuity |
| Speech/emotion models | Natural multimodal input | Translating uncertain affect into justified state changes |
| Human dialogue evaluation | Perceived quality and empathy | Reproducible state-transition and memory-correctness testing |

The gap is therefore not the absence of individual techniques. The gap is the lack of a unified, auditable architecture that connects them while preserving game control and measurable long-term continuity.

## 5. Potential research gaps

### Gap 1: No unified long-term NPC state model

Existing work commonly focuses on one or two state types: memories, persona, emotion, or goals. There is limited agreement on how these states should interact.

**Required contribution:** define explicit boundaries between immutable persona, current goals, mood, relationship state, episodic events, semantic beliefs, and procedural actions.

### Gap 2: Persona drift under memory updates

Memory systems can cause a model to overfit to recent dialogue. A temporary mood or a model-generated reflection may incorrectly modify the character's identity or values.

**Required contribution:** use a versioned persona specification and gate all proposed state changes. Memory can influence beliefs and relationships, but cannot rewrite protected persona facts without an authored transition.

### Gap 3: Unreliable memory formation and contradictions

A dialogue model may store an inference as if it were a fact, duplicate memories, or preserve mutually inconsistent beliefs. Retrieval systems generally rank memories but do not fully determine which belief should remain authoritative.

**Required contribution:** attach provenance, confidence, timestamps, source events, visibility, and supersession links to memories. Distinguish observed events from interpretations and reflections.

### Gap 4: Emotion detection is disconnected from appraisal

Many systems detect a player's emotion and pass the label directly to generation. This ignores the NPC's goals, values, expectations, and relationship history.

**Required contribution:** implement an appraisal step that combines player affect evidence with event meaning, NPC goals, social norms, and relationship context. Confidence should scale the magnitude of state updates.

### Gap 5: Relationship variables lack semantic and temporal grounding

Simple friendship or trust scores are easy to implement but difficult to interpret. A player may be trusted to keep a promise but not trusted with a secret.

**Required contribution:** use multidimensional, relationship-specific state such as trustworthiness, affinity, respect, fear, obligation, and perceived reliability, each linked to evidence and decay or reinforcement rules.

### Gap 6: Dialogue quality is separated from game action validity

An NPC may say that it gave an item, accepted a quest, or changed its mind without the game state reflecting that claim.

**Required contribution:** generate structured response proposals containing speech, emotion display, and typed game actions. Validate actions against authoritative game state before execution and produce speech from the accepted result.

### Gap 7: Limited longitudinal evaluation

Many papers test individual turns, short conversations, or task completion. This is insufficient for a system whose main benefit is continuity over days or weeks.

**Required contribution:** create replayable longitudinal scenarios with repeated players, delayed consequences, broken promises, contradictory evidence, and session boundaries.

### Gap 8: Local-model constraints are underexplored

Large hosted models can hide latency, cost, and context-window assumptions. A local model must work within limited compute, memory, and response-time budgets.

**Required contribution:** evaluate retrieval compression, quantization, context allocation, structured decoding, and fallback behavior under an explicit latency and hardware budget.

### Gap 9: Uncertainty is not propagated through the pipeline

Speech recognition, intent classification, emotion recognition, memory extraction, and generation all contain uncertainty. Most architectures do not preserve it across stages.

**Required contribution:** carry confidence values through the pipeline and use them in memory creation, appraisal strength, relationship updates, and response validation.

### Gap 10: Privacy and player agency are insufficiently addressed

Persistent memories may contain sensitive player information. Emotional adaptation can also become manipulative if it is not transparent and bounded.

**Required contribution:** define memory scope, retention, deletion, player visibility, consent, and safety rules as system requirements rather than afterthoughts.

## 6. Proposed research contribution

The project can make a defensible contribution by implementing and evaluating a **provenance-aware, appraisal-driven, persona-constrained NPC architecture** for local language-model dialogue.

The contribution consists of five integrated mechanisms:

1. **Layered state model:** separates canonical persona, world facts, goals, mood, relationships, episodes, beliefs, and actions.
2. **Provenance-aware memory:** records the source, confidence, timestamp, scope, and status of every memory or belief.
3. **Appraisal-driven adaptation:** converts player actions and uncertain affect into interpretable mood and relationship updates.
4. **Validated structured generation:** requires the local model to propose speech and game actions in a schema checked by deterministic rules.
5. **Longitudinal evaluation:** measures continuity, consistency, state accuracy, action validity, latency, and player-perceived believability over repeated sessions.

The novelty is primarily **architectural and evaluative**: the project integrates established methods into a controlled NPC system and tests whether the integration improves long-term consistency compared with simpler baselines.

## 7. Research questions

### Primary research question

How effectively can a provenance-aware, appraisal-driven memory architecture improve the long-term persona consistency and relationship continuity of a local-LLM NPC?

### Secondary questions

1. Does layered episodic-semantic memory reduce fabricated or contradictory NPC memories compared with recent-context-only prompting?
2. Does appraisal-based state updating produce more believable emotional adaptation than direct emotion-label prompting?
3. Do persona constraints reduce persona drift without making responses repetitive or unnatural?
4. Does structured action validation reduce dialogue-world inconsistencies?
5. What is the effect of retrieval strategy, memory consolidation, and context size on local-model latency and response quality?
6. How do players perceive continuity when relationship state is expressed through dialogue, actions, and future recall?

## 8. Testable hypotheses

**H1 — Memory continuity:** NPCs using episodic-semantic retrieval will recall relevant prior events more accurately than NPCs using only the recent conversation window.

**H2 — Persona consistency:** Protected persona constraints will reduce contradictions with authored backstory and traits compared with unconstrained generation.

**H3 — Emotional plausibility:** appraisal-driven updates will produce higher human-rated emotional appropriateness than direct player-emotion conditioning.

**H4 — World consistency:** schema validation and authoritative action checks will reduce invalid game-state claims compared with free-form response generation.

**H5 — Player experience:** longitudinal interaction with persistent relationships will increase perceived NPC believability and continuity, provided that memory retrieval precision remains above a defined threshold.

## 9. Proposed evaluation design

### Baselines

Compare at least these configurations:

1. scripted dialogue or finite-state baseline;
2. local LLM with recent conversation only;
3. local LLM with persona prompt and recent context;
4. local LLM with retrieved memory;
5. full architecture with memory, appraisal, relationships, persona constraints, and action validation.

### Scenario categories

Use controlled scenarios covering:

- first meeting and persona introduction;
- repeated help and trust accumulation;
- insult, apology, and emotional repair;
- promise, delayed fulfillment, and broken promise;
- contradictory player claims;
- ambiguous or low-confidence speech emotion;
- secret sharing and relationship-specific trust;
- world-state conflict, such as an unavailable item;
- session interruption and later re-entry;
- prompt injection or attempts to rewrite the NPC's identity.

### Metrics

| Dimension | Measurement |
| --- | --- |
| Memory accuracy | Precision, recall, contradiction rate, and source attribution |
| Persona consistency | Backstory/trait violation rate and blinded human ratings |
| Relationship continuity | Agreement between expected and actual state transitions |
| Emotional appropriateness | Human ratings plus appraisal/state consistency checks |
| Action validity | Accepted action percentage and invalid-claim rate |
| Retrieval quality | Recall of relevant memories and irrelevant-memory rate |
| Longitudinal stability | Performance across sessions and delayed consequences |
| Runtime performance | p50/p95 end-to-end latency, memory use, and model throughput |
| Player experience | Believability, immersion, agency, trust, and repetitiveness |

### Ablation studies

Remove one component at a time:

- no semantic consolidation;
- no provenance;
- no appraisal;
- no relationship dimensions;
- no persona validator;
- no action validator;
- no confidence propagation.

This shows which components cause measurable improvement instead of attributing all gains to the language model.

## 10. Limitations and threats to validity

1. **Model dependence:** results may change with the local language model, quantization, prompt format, and hardware.
2. **Scenario dependence:** a small set of scripted scenarios cannot represent every social interaction.
3. **Emotion-label ambiguity:** human annotators may disagree about player affect and appropriate NPC responses.
4. **Novelty effects:** players may initially prefer any generative NPC because it is new.
5. **Evaluation leakage:** an automated judge may reward fluent responses without detecting subtle persona or relationship errors.
6. **Memory sparsity:** short experiments cannot establish whether the architecture remains reliable over months.
7. **Game-domain specificity:** results from one game world may not transfer to another genre or player population.
8. **Privacy constraints:** collecting longitudinal interaction data requires explicit consent, minimization, and retention controls.

These limitations should be reported with the results rather than hidden behind a single aggregate score.

## 11. Final analytical conclusion

The literature provides strong individual foundations for believable agents, dialogue grounding, retrieval, memory, appraisal, planning, speech processing, and evaluation. However, the fields remain fragmented. Existing work does not fully specify how a local language model should combine persistent memories, protected persona facts, mutable relationships, uncertain affect, game-world actions, and auditable state transitions over long-term interaction.

The project is therefore justified if it is framed as a controlled integration and evaluation problem rather than as the invention of every underlying technique. Its strongest research value is a measurable architecture that makes NPC adaptation persistent without allowing the language model to become the uncontrolled authority over identity, memory, emotion, or game state.
