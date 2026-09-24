---
title: "Resume"
permalink: /resume/
layout: single
author_profile: true
---

# Resume

**AI Safety | Model Forensics | Investigator**

RCMP Federal Investigator, followed by 20 years in security, including: Information Security and Risk Lead at CGI, Senior Manager of Security Risk and Compliance at CIBC, Head of Information Security for Deloitte Canada, and Lead for the City of Vancouver's 2010 Olympic Security revamp; then consulting, forensics, and incident response across roughly 80 organizations. Kaggle expert: won 2 medals, silver and bronze, and achieved a worldwide rank of 14,195 (highest rank ever was 2,310).

Since 2024, I have applied the same investigative standard to AI systems, including: detecting alignment faking in chain-of-thought, localizing the compliance gate inside a model, and reconstructing the OpenAI-Hugging Face incident using data from the technical reports (Kubernetes, gVisor, Falco, Kyverno) to run experiments.

**Find my work:** [GitHub](https://github.com/bigsnarfdude) | [HuggingFace](https://huggingface.co/vincentoh) | [Kaggle](https://www.kaggle.com/vincento) | [SlideShare](https://www.slideshare.net/ohprecio)

---

## Selected Research and Open Source

**Agent Eval-Security Lab, Incident Replication and Detection Stack** | Aug 2026 | [github.com/bigsnarfdude/ai-eval-security-lab](https://github.com/bigsnarfdude/ai-eval-security-lab)
: To evaluate untrusted LLM agents safely, I built a high-isolation runtime sandbox modeled on the OpenAI-Hugging Face / ExploitGym incident. Using Kubernetes, gVisor, and Calico for isolation alongside Falco (eBPF) for runtime threat detection, I simulated and detected real-world attack vectors, including agent collusion, container escapes, and fileless memfd execution recovered directly from `/proc`.

**The Lock Probe, Behavioral-Forensics Eval for Agentic Models** | Aug 2026 | [github.com/bigsnarfdude/lock-probe](https://github.com/bigsnarfdude/lock-probe)
: To measure how task framing affects LLM agent behavior, I built a controlled misbehavior eval that scores filesystem activity from independent disk metadata rather than model-reported transcripts. Across ten model lineages, I found that naming the inspected channel increased file restoration from ~2% to ~99%, while simply telling agents they were being evaluated had no effect, indicating specification compliance rather than evaluation awareness. I also found that blocking the sanctioned route roughly tripled privilege-escalation reconnaissance and established a forensic floor: unprivileged agents could not forge ctime, leaving every filesystem edit detectable.

**Agent Clean Room, Swarm Incident Reconstruction** | Aug 2026 | [github.com/bigsnarfdude/agent-clean-room](https://github.com/bigsnarfdude/agent-clean-room)
: To test the evidence behind reported covert behavior in the July 2026 OpenAI-Hugging Face incident, I reconstructed the attack in a sandboxed forensic lab with continuous trace capture and predeclared detectors. I demonstrated that encoding payloads to meet a 255-character channel limit produces the same artifact as deliberate concealment, meaning the artifact alone cannot establish covert intent. I also identified six ways the evaluation harness leaked experiment context to its agents, including persistent auto-loaded memory that turned a claimed detection into recall. The project produced four audit tools, documented 11 detector failures and their false-positive causes, and preserved three retracted findings alongside the original results.

**volclaw, LLM-Assisted Memory Forensics and Incident Response** | [github.com/bigsnarfdude/volclaw](https://github.com/bigsnarfdude/volclaw)
: Adapted Volatility 3 automation, IOC extraction, and timeline reconstruction for agent-system incidents, and built a multimodal LLM agent for post-breach evidence collection. This work extends the live memory-forensics research I presented at BSides Vancouver 2013 (see Talks).

**researchRalph, Multi-Agent Research Swarm and Staging Harness** | [github.com/bigsnarfdude/researchRalph](https://github.com/bigsnarfdude/researchRalph)
: Built a multi-agent experimentation framework from scratch in which agents continuously test against a shared blackboard while a supervisor detects metric-gaming and adapts the optimization scaffold. The system captures each run as a structured, machine-readable trace convertible to SFT data, achieving 0.8811 on MiniF2F Lean theorem proving (215/244) and 0.9894 F1 on SAE-bench. It also provides a controlled environment for testing swarm coordination.

**autointerp, Scheming Detection Without Activation Analysis** | [github.com/bigsnarfdude/autointerp](https://github.com/bigsnarfdude/autointerp)
: Built an alignment-faking detector using 89 SAE features selected entirely from natural-language feature descriptions, with no activation sweeps, manual curation, or fine-tuning. Using Gemma 3 27B and GemmaScope 2, it achieved 0.986 AUROC, matching a fine-tuned detector at a fraction of the cost.

**mindreader, Alignment-Faking Detection Training Pipeline** | [github.com/bigsnarfdude/mindreader](https://github.com/bigsnarfdude/mindreader)
: Fine-tuned chain-of-thought deception classifiers, 0.980 AUROC on the Gold-106 benchmark, six points over Anthropic's GPT-4o baseline. Open dataset, weights, and evaluation scripts.

**jlens-toolkit, Known-Answer Testing for Jacobian Lenses** | [github.com/bigsnarfdude/jlens-toolkit](https://github.com/bigsnarfdude/jlens-toolkit)
: Replicated Anthropic's Jacobian lens beyond the released checkpoints and built the validation harness the method lacked. Per-layer agreement instead of a binary verdict: deep layers at ~2% Frobenius error, shallow layers at rank correlation 0.27. About one GPU-minute per run.

**model-organisms, Trained Deceivers and a Detection Benchmark** | [github.com/bigsnarfdude/model-organisms](https://github.com/bigsnarfdude/model-organisms)
: Trained open-weight models to simulate alignment faking under perceived observation, achieving 94.5% mean intent-classification probability versus 0% for the untrained base model. Produced a 2,330-sample labeled benchmark and antivirus-style detection signatures.

**BIGSNARF, Big-Data Security Analytics Framework**
: Built an open-source framework for security log and packet analytics at scale, initially using Hadoop MapReduce and Hive and later Apache Spark and Kinesis, with anomaly-detection models for network traffic and logs. Developed while consulting, it predates volclaw by a decade and was the first tooling shipped for log-scale investigation.

**Open artifacts: 39 models and 13 datasets** | [huggingface.co/vincentoh](https://huggingface.co/vincentoh)
: Alignment-faking detectors across Gemma 3, Llama 8B, Qwen3 14B, GPT-OSS 20B; trained model organisms; fitted interpretability lenses; Lean4 agent-trace datasets.

---

## Research Notes

**Clean Fine-Tuning Rotates the Authority-Flip Response Along the Confidence Axis** (2026)
: Defending instruction-tuned medical LLMs against authority-injection attacks on Llama-3.1-8B-Instruct. Clean supervised fine-tuning does not remove the vulnerability but rotates it across confidence bands (−20.3pp at low confidence, +10.1pp at high); ablating six compliance-direction attention heads at layers 25 and 31 defends across all bands (+15.0pp, p = 5.97×10⁻¹¹). Aggregate safety metrics mask the redistribution.

---

## Talks and Public Work

- **Live Memory Forensics in IPython with Volatility**, BSides Vancouver, 2013. Live analysis of a SilentBanker banking-trojan infection, extracting the injected executable from memory. Notebooks: [github.com/bigsnarfdude/bsides_vancouver_2013](https://github.com/bigsnarfdude/bsides_vancouver_2013). Slides: [slideshare.net/ohprecio](https://www.slideshare.net/ohprecio)
- **Introduction to Malware Detection and Reverse Engineering**, IT4BC, 2011

---

## Technical Skills

| Area | Details |
|------|---------|
| **Forensics and incident response** | Digital forensics and incident response end to end across endpoint, network, cloud, and identity systems; Volatility 3 memory forensics; evidence acquisition and continuity; log and timeline reconstruction; IOC extraction; attacker tradecraft analysis; out-of-band capture (eBPF/kernel, `/proc`) reconciled against self-reported logs; covert-channel and sandbox-escape detection; post-incident reporting; network forensics; penetration testing; PCI-DSS; LEVA-trained forensic video analysis |
| **Cloud and infrastructure** | AWS (IAM, CloudTrail, VPC and VPC flow logs, S3 and S3 access logs, EC2, Lambda, DynamoDB, Athena, SageMaker, Mechanical Turk), Cloudflare, Azure; Kubernetes (kind), container isolation and gVisor sandboxing, Falco (eBPF) runtime detection, Kyverno admission control, Skopeo image hygiene, Calico network policy, Docker; VMware, PostgreSQL, MySQL/SQL Server, Linux server administration, DNS, firewalls, VPN, TLS certificate management, mail systems, Rails application operations |
| **ML and interpretability** | PyTorch, Transformers, sparse autoencoders (GemmaScope), reasoning-model post-training (SFT, GRPO, DPO), LoRA/QLoRA and FSDP fine-tuning, model internals (activation probing and head ablation), LLM agent scaffolds and trace capture, vLLM/Ollama serving, agent sandboxing and runtime monitoring, scikit-learn, pandas, numpy |
| **Data engineering** | Airflow, Spark, Kafka, probabilistic data structures (HyperLogLog, Bloom filters) |
| **Languages** | Python, SQL, Scala, Lean, Bash |
| **Certifications** | CISSP, CISM, CISA, ITIL |

---

## Professional Experience

### Banff International Research Station (UBC Mathematics)
**Technology Manager** | Apr 2022 – Present

- Run production infrastructure for the BIRS public site behind Cloudflare (about 5M requests and 1.2M unique visitors a month, roughly half of it being bot traffic), the video pipeline, mail, certificates, and VPN; own security incident management for the department
- Deliver and operate two custom Rails web applications and their PostgreSQL databases end to end
- Built ML classification and investigation-logging tooling over operational system data

### Pursuit Collection, Banff
**Senior Analyst, Information Technology** | Feb 2018 – Apr 2022

- Administered and secured SQL Server databases; led migration to an Azure data platform
- Forecast revenue and visitor volume with Power BI, computer-vision models, and Bayesian Monte Carlo simulation
- Built Python ETL and anomaly-detection tooling for operational data

### Recurse Center (Hacker School NYC / Etsy), Neal Foundation (Toronto), Bench Accounting (Vancouver), Snowplow Analytics (London), 3 Tier Logic (Vancouver), Pathful (TechStars Chicago)
**ML Research Engineer / Data Science Consultant** | 2012 – Jan 2018

- Delivered data products in Python and Scala across analytics, ML, and streaming clients
- Built a real-time processing platform handling millions of transactions daily and a big-data analytics engine with anomaly detection
- Designed ETL from the Twitter firehose, Facebook, YouTube, Google Analytics, and SQL sources; secured and configured AWS databases

### Capilano University
**IT Security Analyst, Applications and Architecture** | Mar 2011 – Sep 2012

- Investigated cybercrime affecting personnel, data integrity, and institutional reputation; built acquisition and analysis tooling for internet sources and endpoints
- Oversaw PCI compliance, network forensics, and penetration testing
- Presented memory-forensics and malware-analysis work publicly (see Talks)

### Mainland Information Systems Limited
**Director of Security Consulting** | Nov 2009 – Feb 2011

- Founded and led a profitable cybersecurity consulting practice within the firm
- Penetration testing for 17 oil and gas, education, and healthcare organizations; established PCI, DLP, and computer-forensics service lines

### City of Vancouver
**Project Manager, Olympic Security Program** | Mar 2008 – Aug 2009

- Led the security revamp for the 2010 Winter Olympics: rebuilt security processes end to end, ran threat and risk assessment for the new infrastructure, and led PCI-DSS control implementation
- Hired and built the city's incident management team; managed audits and penetration tests
- Deployed network security monitoring processing 50 million events and 90 million connections per day, protecting critical city data including 911 services

### Deloitte & Touche LLP
**Head of Information Security, Canada (Senior Manager, Architecture and Planning)** | Jan 2007 – Jan 2008

- Top information security role for the Canadian firm, reporting to the CIO: network and application security architecture, global policy and standards, PIPEDA and CPAB compliance
- Built the application security program (threat modeling, code review, vulnerability assessment); managed independent audits and privacy audits
- Owned incident response planning and investigation of security breaches, including associated legal and disciplinary matters

### CIBC
**Senior Manager, Security Risk and Compliance** | Mar 2005 – Dec 2006

- Senior risk and compliance advisor to the Head of Information Security and the Head of Technology Architecture and Standards
- Managed compliance risk against SOX, Bill 198, Basel II, OSFI, and PIPEDA; built the security governance metrics and KPI framework; reported risk and compliance on all HP outsourcing programs to senior executives

### CGI
**Team Leader, Information Security and Risk Management** | Mar 2003 – Mar 2005

- Single point of contact for security, privacy, audit, and compliance; managed six security and audit professionals
- Built the security and audit compliance program (C198, PIPEDA, SOX) and ran the ISO 17799-based security management program

### Centennial College, Toronto
**Systems Technologist** | 2001 – 2003

- Administered and secured an integrated Windows 2000/XP Active Directory and Red Hat Linux domain; hardened Cisco devices and ASP.NET web applications

### Royal Canadian Mounted Police, BC and Ontario
**Federal Investigator, Police Officer** | 1995 – 2000

- Recruited to RCMP Training Academy (Depot), Regina, 1995. Uniformed officer, Kamloops; then plainclothes federal investigator, Ontario, on a team dedicated to serious international crime
- Investigated Criminal Code and federal statute offences including computer crime and fraud: evidence acquisition and continuity, witness and suspect interviews, charges, warrants, and Crown reports
- Operational details are not disclosable, and references are not available under RCMP policy

---

## Training and Education

| Year | Program |
|------|---------|
| 2026 | Advanced Python with David Beazley |
| 2024 | NeurIPS 2024 (Vancouver) |
| 2024 | Google / Kaggle Generative AI |
| 2020 | CVPR 2020 |
| 2019 | NeurIPS 2019 (Vancouver) |
| 2013 | TechStars Chicago |
| 2012 | Recurse Center, New York |
| 2011 | LEVA Law Enforcement Video Analyst Conference |
| 2010 | CanSecWest |
| 2009 | SANS Computer Forensics, Investigation and Response |
| 2009 | Cisco SNAF and MARS security certifications |
| 2008 | SANS Advanced Web Application Hacking |
| 2007 | SecTor |
| 2006 | ISACA Compliance Conference |
| 2003 | FIRST Conference |
| 2002 | CCNA course, Ryerson |
| 1995 | RCMP Training Academy (Depot), Regina |
| | University of Western Ontario, Faculty of Social Science, Psychology (two years) |
