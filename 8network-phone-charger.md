# Resonance: Collective Attention Telemetry

8 network phone charger {bottle brewery narrative testing for user guided 'reality' construction}

## Vision

Resonance is an instrument for observing how meaning forms collectively across media boundaries in real time. It detects when independent users exhibit correlated semantic behavior within shared temporal windows, surfacing those correlations as meaningful context without surveillance or individualized profiling.

This is not a recommendation engine, social feed, or sentiment analyzer. It is a **semantic resonance detector**—a real-time map of distributed cognition.

---

## Core Concepts
![private interest cars](https://github.com/user-attachments/assets/45b2d330-aa29-4388-8c56-7e88fe677e87)

### Resonance Event
<img width="777" height="516" alt="Screenshot 2025-12-25 at 7 46 16 PM" src="https://github.com/user-attachments/assets/b49a2ad6-ebad-44f9-bbd5-a9b1fe01026a" />



A Resonance Event is formally defined as a tuple:

```
R = (T, U, D, S, Δ)
```

| Component | Definition |
|-----------|------------|
| **T** | Time window (sliding: 60s, 5min, 30min) |
| **U** | Number of distinct anonymous users |
| **D** | Distinct domains/content types involved |
| **S** | Semantic cluster in embedding space |
| **Δ** | Deviation from historical baseline for this pattern |

This allows us to measure:
- **Resonance bandwidth**: how many distinct domains and behaviors are involved
- **Resonance amplitude**: how far from baseline
- **Resonance trajectory**: how the cluster moves through semantic space over time

### Epistemic Registers

Events are classified across parallel interpretive layers:

| Register | Indicators |
|----------|------------|
| **Expert sensemaking** | GitHub, arXiv, standards docs, specialized forums |
| **Consumerization** | Shopping, TikTok, mainstream news, short-form video |
| **Governance/Legal** | Regulations, legal analysis, policy documents |
| **Primary sources** | Academic papers, official statements, raw data |

Tracking how an event propagates across registers reveals how meaning is being constructed by different epistemic communities.

### Semantic Axes (Not Sentiment)

Instead of positive/negative sentiment, we track movement along interpretive dimensions:

- **Skeptical ↔ Trusting**: critical blogs vs industry sources
- **Technical ↔ Surface**: specs and docs vs headlines and explainers  
- **Local ↔ Global**: regional news vs international policy bodies
- **Emerging ↔ Established**: novel framing vs canonical interpretations

---

## Architecture

### Layer 0: Client (Browser Extension)

**Principle**: Maximum local processing, minimum data transmission.

#### Data Collected (Events, Not Identity)

- Page domain + normalized URL category
- Timestamp (coarsened to 30-60s buckets)
- Semantic fingerprint (title, meta description, extracted entities)
- Interaction type: open/close, tab focus, dwell time bucket
- Interaction semantics: revisit patterns, rapid switching, skim vs deep read

#### Data Never Collected

- Full URLs with parameters
- Keystrokes or form inputs
- Cross-session persistent identity
- Raw page content

#### Local Processing Pipeline

```
Raw page → Entity extraction (YAKE/TextRank)
        → Topic sketching (lightweight biterm model)
        → Embedding (MiniLM/E5-small via WASM)
        → Hash + compress
        → Transmit only: embeddings, entity hashes, topic IDs, interaction type
```

#### Trust Posture

- UI panel showing last N events queued for transmission
- Per-signal-type toggles
- "Panic off" switch
- Open-source, auditable codebase

### Layer 1: Ingestion & Anomaly Detection

**Statistical baseline and burst detection.**

```
Events → Kafka/Redpanda → ClickHouse (columnar storage)
                       → Baseline computation (hourly/daily by semantic bucket)
                       → Z-score / EVT anomaly flagging
```

Thresholds:
- Minimum U (users) per window
- Minimum D (domains) per cluster
- Minimum Δ (deviation) from baseline

### Layer 2: Multi-Layer Graph Clustering

**The system's validity hinges on distinguishing collective cognition from coordinated noise.**

The core challenge: a spike in "Nike" searches during a basketball game is obvious. The system's value is finding *non-obvious, cross-domain semantic bridges*.

#### Architecture: Temporal Knowledge Graph Distillation

```
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 1: EVENT NODES                                           │
│  Raw events from extensions (embedding + entities + domain)     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 2: CO-OCCURRENCE EDGES                                   │
│  Connect events within window W1 (2 min) if cosine sim > T1     │
│  Forms many small, dense "moment graphs"                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 3: USER-AGNOSTIC MERGE                                   │
│  Merge moment graphs across users within W2 (10 min)            │
│  Community detection (Leiden/Louvain) on merged graph           │
│  Each community = candidate semantic resonance                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 4: CROSS-DOMAIN FILTER & AMPLIFICATION                   │
│  Signal = f(community_size, domain_diversity, novelty_score)    │
└─────────────────────────────────────────────────────────────────┘
```

#### Signal Scoring Function

```
Signal(cluster) = size_weight × log(U) 
                + diversity_weight × D/D_max
                + novelty_weight × (1 - baseline_overlap)
                - homogeneity_penalty × referrer_concentration
```

Where:
- **U**: unique users in cluster
- **D**: distinct domain types (require ≥3: e.g., video, encyclopedia, e-commerce, code)
- **baseline_overlap**: entity similarity to rolling 24hr baseline (suppress evergreen)
- **referrer_concentration**: penalty if >70% events share single referrer

#### Why Graph-Based?

Graphs naturally surface **bridge concepts**. A single event might link two emerging clusters, revealing deeper synthesis—e.g., a philosophical Wikipedia page connecting a political livestream cluster to a fantasy novel cluster. These bridges are often the most valuable insights.

#### Outputs

- Cluster membership over time
- Bridge nodes connecting clusters
- Migration patterns (e.g., "entertainment → legal analysis")
- Cross-domain correlation signatures

### Layer 3: LLM Narrativization

**Summarization and labeling only—never raw ingestion. The LLM's one right job is narrative inference, not pattern detection.**

Input: sampled titles, hashed topic labels, cluster statistics

#### Prompt Framework for Cluster Labeling

```
Context: During the time window {window}, {user_count} anonymous users 
collectively browsed content across {domain_count} distinct types of 
sites (e.g., video, shopping, forums).

The strongest shared concepts were: {entity_list}.

The top representative page titles were:
- {title_1}
- {title_2}
- ...

Task: Generate:
1. A **neutral, descriptive hypothesis** for what collective activity 
   or inquiry was likely occurring.
2. The **most likely catalyst** (e.g., "a news event," "a viral video," 
   "a product launch").
3. A **"Why this matters"** statement focusing on the cross-domain 
   pattern, not the content itself.

Constraints:
- Do not state assumptions as facts.
- If the link is unclear, state the ambiguity.
- Focus on the pattern of migration (e.g., "from entertainment to 
  technical documentation").
```

#### Example Output

> *"A group appears to be moving from a martial arts livestream to Chinese history pages and then to weapon replicas shopping sites, suggesting a moment where historical reference is driving consumer interest."*

#### Tasks

| Task | Output |
|------|--------|
| **Naming** | Short neutral labels ("AI copyright disputes", "GPU supply anxiety") |
| **Change summary** | "Compared to baseline, movement from consumer news → legal analysis, suggesting shift from awareness to rule interpretation" |
| **Phase inference** | Emergence, contestation, consolidation, decay |
| **Catalyst hypothesis** | Most likely trigger event category |
| **Ambiguity flagging** | Explicit uncertainty when links are unclear |

### Layer 4: Symbolic Structure (Optional/Advanced)

**Event graphs for rule-based queries.**

```
Cluster → Event graph (nodes: topics, domains, intent tags; edges: temporal transitions)
       → Query engine for patterns:
          - "Expert concern emerging?" → mass media → specialized forums → preprints
          - "Fragmentation?" → multiple disjoint interpretations, similar amplitude
```

This enables moving beyond trend detection toward observing collective reasoning.

---

## Privacy & Legitimacy Architecture

### Threat Model

Assume a strong attacker controlling the backend. Design so that even this attacker cannot reconstruct individual user journeys.

### Structural Protections

| Protection | Implementation |
|------------|----------------|
| No persistent user IDs | Rotating anonymous session keys |
| Differential privacy | Noise injection on all counts |
| Aggregation thresholds | No output for clusters < N users |
| Secure aggregation | Multi-party computation for sensitive co-occurrence stats |
| Data minimization | Delete raw events after clustering; retain only aggregates |

### Transparency Hooks (User-Facing)

Every insight displays:
- "How many users contributed to this?"
- "What data types were used?"
- "Confidence level"
- "View the cluster's semantic neighbors"

### Hard-Coded Constraints

- No law enforcement use
- No individualized profiling
- No political microtargeting
- No sale of individual-level data
- Public data schema

---

## Failure Modes & Mitigations

Critical threats to signal integrity and ethical operation:

### 1. The Viral Trap

**Threat**: A single influencer saying "Go look this up" creates a massive, artificial signal that mimics organic resonance.

**Detection**:
- Track referrer source homogeneity within clusters
- Flag if >70% of cluster events originate from single referrer domain in 30-second burst

**Mitigation**:
- Label as "potentially directed" rather than organic resonance
- Reduce signal score via `referrer_concentration` penalty
- Surface referrer source in transparency metadata

### 2. Platform Bias

**Threat**: Early adopters will skew tech/Reddit, making resonance appear in niche topics that don't represent broader attention.

**Detection**:
- Track user population demographics (opt-in, aggregated)
- Compare cluster topic distribution against external baselines (Google Trends, etc.)

**Mitigation**:
- Explicitly report **estimated cultural basin** of each signal
- Label: "Global Internet," "Tech Twitter," "Academic Finance," etc.
- Be transparent that perspective is limited to participating population

### 3. Ghost Clusters

**Threat**: Ad networks serving identical content across sites create false semantic resonance—same ad/article appears on multiple domains, looks like organic cross-domain interest.

**Detection**:
- Fingerprint common commercial content (ad copy, syndicated articles)
- Maintain server-side blocklist of known content distribution network patterns

**Mitigation**:
- Suppress events matching known CDN/syndication fingerprints
- Require semantic diversity beyond surface-level title matching
- Weight original content sources higher than aggregators

### 4. Weaponization Risk

**Threat**: Insights like "people watching this political debate are suddenly researching gun laws" could be dangerous in wrong hands or wrong contexts.

**Detection**:
- Classify clusters by topic sensitivity (politics, health, finance, security)
- Flag high-risk topic combinations automatically

**Mitigation**:
- **Human-review gate** for clusters involving high-risk topics before public display
- Err on the side of silence—withhold ambiguous high-risk signals
- No real-time feeds for sensitive categories; minimum delay for review
- Audit log of all suppression decisions

### 5. Coordination Attacks

**Threat**: Bad actors intentionally create artificial resonance patterns to manipulate the system's outputs (astroturfing attention).

**Detection**:
- Statistical anomaly detection on user behavior patterns
- Flag suspiciously synchronized event timing
- Monitor for bot-like navigation patterns (no dwell time variance, mechanical transitions)

**Mitigation**:
- Require minimum behavioral entropy per contributing user
- Weight long-term consistent users higher than new accounts
- Rate-limit influence of any single session on cluster formation

---

## UX Surfaces

### A. Live Contextual Overlays

While consuming content:
- "People engaging with this are currently exploring..."
- "Related concepts spiking right now"
- "Parallel interpretations emerging in [expert/consumer/governance] registers"

### B. Collective Footnotes

For articles and videos:
- Community-derived reference trails
- Not comments—evidence of where attention flows after this content
- Epistemic register breakdown

### C. Resonance Dashboard

For researchers/journalists:
- Timeline with semantic clusters rising/falling
- Heatmap of cross-domain correlations
- Replay mode: scrub through an event's lifecycle
- Epistemic health indicators:
  - Topic fragmentation score
  - Primary vs secondary source reliance
  - Speed of expert engagement
  - Persistence of outdated frames

### D. Contrastive Overlays (Opt-In)

If users volunteer obfuscated traits ("developer", "educator", "general"):
- "Developers converging on protocol specs; general audience on explainers"
- Strict aggregation thresholds; no small-group exposure

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Extension** | Manifest V3, WASM NLP (MiniLM), IndexedDB buffer |
| **Ingestion** | Kafka / Redpanda |
| **Event storage** | ClickHouse |
| **Embeddings** | Qdrant / Weaviate |
| **Graph storage** | Neo4j / NetworkX (for smaller scale) |
| **Processing** | Python (FastAPI + Celery) |
| **Clustering** | HDBSCAN, Leiden/Louvain (community detection), scikit-learn |
| **LLM layer** | Claude API (summarization only) |
| **Frontend** | React + D3 for dashboards |

---

## Development Phases

### Phase 0: Viability Proof (Week 1)

**Goal**: Prove you can detect *any* genuine, non-obvious cross-domain resonance with 5 friends.

#### Extension Skeleton (2 days)

```javascript
// Manifest V3
// Permissions: activeTab, storage, host permissions for test sites
// Target sites: YouTube, Wikipedia, Amazon, GitHub

// On tab update:
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete') {
    const event = {
      session_id: getRotatingSessionId(),
      timestamp: Date.now(),
      domain: new URL(tab.url).hostname,
      title: tab.title,
      topic_vector: extractTopicVector(tab.title, tab.url)
    };
    queueEvent(event);
  }
});

// Batch send every 30s
setInterval(() => {
  if (eventQueue.length > 0) {
    fetch('http://localhost:8000/events', {
      method: 'POST',
      body: JSON.stringify(eventQueue)
    });
    eventQueue = [];
  }
}, 30000);

// Topic vector: crude TF-IDF on title + domain keywords
function extractTopicVector(title, url) {
  const words = tokenize(title + ' ' + url);
  return words.filter(w => w.length > 3).slice(0, 10);
}
```

#### Local Server (1 day)

```python
from fastapi import FastAPI
from sklearn.cluster import DBSCAN
from sklearn.feature_extraction.text import TfidfVectorizer
import numpy as np

app = FastAPI()
events = []

@app.post("/events")
async def receive_events(batch: list):
    events.extend(batch)
    return {"status": "ok", "total": len(events)}

@app.get("/clusters")
async def get_clusters():
    if len(events) < 10:
        return {"clusters": [], "message": "insufficient data"}
    
    # Vectorize topic vectors
    texts = [' '.join(e['topic_vector']) for e in events]
    vectorizer = TfidfVectorizer()
    X = vectorizer.fit_transform(texts)
    
    # Cluster
    clustering = DBSCAN(eps=0.5, min_samples=3, metric='cosine')
    labels = clustering.fit_predict(X)
    
    # Find cross-domain clusters
    results = []
    for cluster_id in set(labels):
        if cluster_id == -1:
            continue
        cluster_events = [e for e, l in zip(events, labels) if l == cluster_id]
        domains = set(e['domain'] for e in cluster_events)
        if len(domains) >= 3:
            results.append({
                "id": cluster_id,
                "size": len(cluster_events),
                "domains": list(domains),
                "sample_titles": [e['title'] for e in cluster_events[:5]]
            })
    
    return {"clusters": results}
```

#### Coordinated Test (1 evening)

1. Have all 5 participants watch the same obscure documentary clip
2. At an unannounced time, mention a related but non-obvious concept in chat
   - Example: if documentary is about bees, say "I wonder how hive algorithms relate to load-balancing"
3. Do not instruct anyone what to do
4. Monitor `/clusters` endpoint
5. **Success**: Within 10 minutes, a cluster emerges linking:
   - Video domain (YouTube)
   - Wikipedia (bees, algorithms)  
   - GitHub (load-balancing repos)

**That's your first resonance. If that works, you have magic. Then rebuild properly.**

---

### Phase 1: Foundation (Weeks 1-4)

**Goal**: Prove that correlated browsing behavior produces meaningful clusters.

- [ ] Browser extension skeleton (Manifest V3)
  - [ ] Tab open/close/focus event capture
  - [ ] Timestamp coarsening
  - [ ] Local keyword extraction (YAKE)
  - [ ] UI panel showing pending events
- [ ] Minimal backend
  - [ ] Event ingestion endpoint
  - [ ] ClickHouse schema for events
  - [ ] Basic temporal windowing
- [ ] Private alpha deployment (10-20 users)
  - [ ] Shared event: livestream, keynote, or breaking news
  - [ ] Manual inspection of raw event patterns

**Success criteria**: Can observe correlated domain access patterns during a shared event.

### Phase 2: Graph Clustering (Weeks 5-8)

**Goal**: Multi-layer graph clustering that surfaces bridge concepts.

- [ ] Client-side embeddings
  - [ ] MiniLM via WASM
  - [ ] Embedding transmission instead of raw text
- [ ] Graph-based clustering pipeline
  - [ ] Event layer: timestamped nodes with embeddings
  - [ ] Co-occurrence layer: edges for temporal proximity + semantic similarity
  - [ ] Moment graph formation (W1 = 2min windows)
  - [ ] User-agnostic merge across W2 = 10min windows
  - [ ] Community detection (Leiden algorithm)
- [ ] Signal scoring implementation
  - [ ] Domain diversity requirement (≥3 types)
  - [ ] Novelty score vs 24hr baseline
  - [ ] Referrer concentration penalty (viral trap detection)
- [ ] Bridge concept detection
  - [ ] Identify nodes connecting multiple communities
  - [ ] Surface cross-cluster semantic links
- [ ] Basic dashboard
  - [ ] Timeline view of cluster activity
  - [ ] Bridge concept highlighting
  - [ ] Click-through to representative URLs

**Success criteria**: 
- Clusters form and dissolve corresponding to real-world events
- Bridge concepts identified between related clusters
- Viral/directed signals correctly flagged

### Phase 3: Narrativization & Trajectories (Weeks 9-12)

**Goal**: Human-readable insights from cluster dynamics.

- [ ] LLM summarization pipeline
  - [ ] Cluster naming
  - [ ] Change summaries ("compared to baseline...")
  - [ ] Phase labeling
- [ ] Trajectory modeling
  - [ ] Temporal graph construction
  - [ ] Migration pattern detection
  - [ ] Epistemic register classification
- [ ] Enhanced dashboard
  - [ ] Resonance event cards with (T, U, D, S, Δ)
  - [ ] Trajectory visualization
  - [ ] Register breakdown

**Success criteria**: Non-technical users can understand "what is happening" from dashboard outputs.

### Phase 4: Privacy Hardening (Weeks 13-16)

**Goal**: Production-grade privacy architecture.

- [ ] Differential privacy implementation
  - [ ] Noise calibration
  - [ ] Threshold enforcement
- [ ] Secure aggregation exploration
  - [ ] Evaluate MPC options for co-occurrence stats
- [ ] Formal privacy audit
  - [ ] External review of data flows
  - [ ] Threat model documentation
- [ ] Transparency features
  - [ ] Per-insight provenance display
  - [ ] User data export/deletion

**Success criteria**: Pass external privacy review; publish threat model and data schema.

### Phase 5: Public Beta & Surfaces (Weeks 17-24)

**Goal**: Usable product for early adopters with robust signal integrity.

- [ ] Extension polish
  - [ ] Onboarding flow
  - [ ] Granular controls
  - [ ] Performance optimization
- [ ] Signal integrity features
  - [ ] Ghost cluster suppression (CDN/syndication fingerprinting)
  - [ ] Cultural basin labeling ("Tech Twitter", "Academic Finance", etc.)
  - [ ] Human review gate for high-risk topic clusters
- [ ] Live overlay prototype
  - [ ] Browser-side rendering of contextual insights
  - [ ] "Potentially directed" labeling for viral trap signals
- [ ] Collective footnotes prototype
  - [ ] Integration with reading mode or specific sites
  - [ ] Bridge concept highlighting
- [ ] Research dashboard
  - [ ] Replay mode
  - [ ] Epistemic health metrics
  - [ ] Export for academic use

**Success criteria**: 100+ users in open beta; positive feedback from journalists/researchers; <5% false positive rate on viral trap detection.

### Phase 6: Advanced Features (Weeks 25+)

- [ ] Symbolic query layer
  - [ ] Event graph construction
  - [ ] Pattern queries ("expert concern emerging?")
- [ ] Adversarial robustness
  - [ ] Coordination attack detection
  - [ ] Behavioral entropy requirements
  - [ ] Bot pattern recognition
- [ ] Intervention sandbox
  - [ ] Opt-in cohort experiments
  - [ ] Measure effect of overlays on trajectories
- [ ] API for third-party integration
  - [ ] Aggregate-only endpoints
  - [ ] Rate limiting and access controls
- [ ] Cross-community contrastive views
  - [ ] Opt-in trait collection
  - [ ] Comparative overlays

---

## Success Metrics

### Technical

| Metric | Target |
|--------|--------|
| Cluster coherence (internal edge density) | > 0.4 |
| Latency: event → cluster assignment | < 60s |
| False positive rate (noise clusters) | < 10% |
| Bridge concept detection rate | > 50% of cross-domain links surfaced |
| Viral trap detection accuracy | > 90% |
| Privacy: re-identification risk | Negligible under formal analysis |

### Product

| Metric | Target |
|--------|--------|
| Daily active users (beta) | 500+ |
| Insight utility rating | > 4/5 |
| Researcher adoption | 10+ academic/journalism partners |
| User trust score | > 4/5 on privacy perception |

### Impact

| Metric | Target |
|--------|--------|
| Novel event detection | Identify emerging events before mainstream coverage |
| Epistemic health tracking | Publish monthly reports on information ecosystem |
| Media literacy integration | Adoption by 3+ educational institutions |

---

## Open Questions

1. **Cluster granularity**: How do we balance specificity (useful) vs. noise (too many micro-clusters)?

2. **Temporal decay**: How quickly should old events age out of influence on current clusters?

3. **Cross-event linking**: When does a new cluster represent a genuinely new phenomenon vs. a continuation of an earlier one?

4. **Legitimacy signaling**: Beyond technical privacy, how do we build trust with skeptical users who've been burned by "privacy-preserving" products before?

5. **Monetization**: Research grants? Institutional subscriptions? What models preserve the public-interest mission?

6. **Adversarial robustness**: How do we detect and mitigate coordinated attempts to manipulate cluster formation?

---

## Naming Candidates

The **collective footnotes** concept is the killer feature. It's not about what people *think*, but what they *need to know* to understand the moment.

**Core metaphor**: Waze for collective attention—not showing where traffic is, but showing where the mind of the network is flowing and why.

### Primary Candidates

| Name | Framing |
|------|---------|
| **Resonance** | Physics metaphor—synchronized oscillation across independent systems |
| **Contexture** | The texture of context; woven meaning |
| **The Footnote Engine** | What you need to know to understand this moment |

### Secondary Candidates

- Semantic Echo
- Collective Sensorium  
- Attention Field
- Semantic Weather
- The Present Tense
- Distributed Cognition Observatory (DCO)
- Resonance Notes

---

## ADDENDUM: Epistemic Feedback Loop Warning

### The Danger Beyond Privacy

This system, if deployed covertly—particularly if used to inform content generation—would create something far more dangerous than a privacy violation. It would constitute **industrialized narrative experimentation on an unwitting population**.

The mechanism is as follows:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. OBSERVE collective attention patterns in real-time          │
│     (what people look up, how meaning migrates across domains)  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  2. GENERATE content calibrated to those patterns               │
│     (narratives, framings, emphases tuned to observed flows)    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  3. DEPLOY that content into the information environment        │
│     (users encounter it as "organic" news, content, context)    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  4. OBSERVE how collective attention shifts in response         │
│     (which framings gain traction, which die, which mutate)     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
                         [REPEAT]
```

This is not content personalization. This is **real-time A/B testing of narratives about reality** where the subjects don't know they're in the experiment.

### Why This System Makes It Worse

Standard recommendation algorithms optimize for engagement with existing content. This system would enable something qualitatively different: **optimizing the generation of new framings based on how collective sensemaking actually operates**.

The danger scales with capability:

| Capability Level | Risk |
|-----------------|------|
| **Observe only** | Privacy violation; potential for surveillance |
| **Observe + Recommend** | Filter bubble effects; attention manipulation |
| **Observe + Generate** | Narrative injection calibrated to cognitive vulnerabilities |
| **Observe + Generate + Iterate** | Closed-loop epistemic engineering; reality becomes a dependent variable |

At the highest level, you are no longer describing what people believe—you are **instrumenting belief formation itself** as a control system.

### The Feedback Loop Problem

When collective attention data informs generated content without disclosure:

1. **Users cannot distinguish organic from synthetic sensemaking.** A "trend" might be genuine collective inquiry or might be a probe to test which framings propagate.

2. **The system learns what reality-claims gain traction.** Not what's true—what's *sticky*. These are not the same. The optimization target is narrative fitness, not accuracy.

3. **Successful framings get reinforced; unsuccessful ones get modified and re-tested.** The population becomes an unwitting training set for persuasion.

4. **There is no exit.** Users cannot opt out of a system they don't know exists. They cannot adjust for manipulation they cannot perceive. The feedback loop is invisible by design.

5. **Ground truth erodes.** When the information environment is continuously adjusted based on what framings succeed, the distinction between "what people believe" and "what we've trained people to believe" collapses.

### This Is Not Hypothetical

Variants of this loop already exist in:
- Algorithmic content generation informed by engagement metrics
- A/B tested headlines and framings at scale
- LLM-generated content tuned to trending discourse

What this system would add is **semantic depth and cross-domain visibility**—the ability to see not just what's clicked but how meaning migrates, which analogies stick, which bridges form between concepts. That's a richer signal for narrative optimization than click-through rates.

In the wrong hands, this becomes a **cognitive radar** for calibrating influence operations.

### Hard Prohibitions

The following uses are **absolutely prohibited** and represent ethical bright lines:

| Prohibited Use | Reason |
|----------------|--------|
| Covert deployment without informed consent | Violates autonomy; makes users unwitting experimental subjects |
| Using resonance data to inform content generation without disclosure | Creates invisible feedback loop; erodes epistemic autonomy |
| Selling or sharing data with entities that generate content | Launders the feedback loop through third parties |
| Government or political use for narrative management | Instrumentalizes democratic discourse |
| Use by platforms to optimize their own content pipelines | Conflict of interest; platform becomes both observer and manipulator |

### Required Safeguards (If This System Exists At All)

1. **Radical transparency.** Users must know this system exists, what it observes, and who has access to its outputs.

2. **No generative coupling.** Resonance data must never feed into content generation systems, period. The observation layer and the generation layer must be architecturally separated with no bridge.

3. **Public-interest-only outputs.** Aggregate insights may be published for journalism, research, and media literacy—but never sold to entities with content-generation capabilities.

4. **Adversarial auditing.** Independent parties must be able to verify that the feedback loop does not exist—not just that it's "against policy" but that it's technically impossible.

5. **Kill switch doctrine.** If evidence emerges that the system is being used to close the loop—even by third parties using public outputs—the system must be shut down, not reformed.

### The Deeper Problem

This warning is not about this system specifically. It's about the convergence of:
- Real-time collective attention telemetry (this project)
- Large-scale content generation (LLMs)
- Optimization under engagement metrics (platforms)

Any two of these are manageable. All three together, without transparency, is **an epistemic control system**—a machine for manufacturing consensus while maintaining the appearance of organic belief formation.

The reason to build Resonance *correctly*—with consent, transparency, and no generative coupling—is precisely because someone will eventually build it *incorrectly*. The public-interest version establishes norms, demonstrates alternatives, and creates legibility for what the malicious version would look like.

**If you cannot build this system in the light, you should not build it at all.**

---

## References

- Divvi Up: Privacy-preserving telemetry architecture
- HDBSCAN: Density-based clustering for noisy data
- Leiden/Louvain: Community detection in graphs
- MiniLM: Lightweight sentence embeddings
- YAKE: Unsupervised keyword extraction
- Differential privacy: Formal framework for privacy guarantees
- NetworkX: Graph analysis in Python
- Neo4j: Graph database for production scale

---

*This document is a living roadmap. Update as the project evolves.*
