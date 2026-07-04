# 🇲🇽 NexoGob: El Observatorio Digital del Estado

> **"Visualizing the Real Structure of the Mexican State."**

![Concept](https://img.shields.io/badge/Status-Concept%20%2F%20Design-red)

**NexoGob** is an open-source intelligence engine designed to aggregate, map, and analyze the entire digital footprint of the Mexican government. By treating the government's web presence as a mathematical graph, we move beyond simple document search to reveal the *de facto* structure of the bureaucracy, uncovering hidden dependencies, broken administrative chains, and patterns indicative of corruption.

## Current Status (2026-07-04)

NexoGob is a **concept / design document. There is no code in this repository.**

- The repo contains three markdown files (this README, `PRD.md`, and
  `ONTOLOGY.md`) — no crawler, no `docker-compose.yml`, no `frontend/`, no npm
  scripts, and no `CONTRIBUTING.md` or `LICENSE` file yet, despite references
  below.
- The "Getting Started" section describes the **intended** developer experience;
  none of those commands can currently be run.
- Roadmap "Phase 1 (Atlas)" reflects **design work only** — no crawling or
  graph infrastructure has been built.
- A likely implementation path is to integrate with
  [tezca](https://github.com/madfam-org/tezca) (MADFAM's Mexican
  legal/compliance corpus platform) for legal-entity and official-source data,
  rather than building all ingestion from scratch.

Everything below is **vision/roadmap** for the system as designed.

-----

## 🧐 The Problem

The Mexican government operates across **5,000+ domains** split between the Federal Executive (`gob.mx`), Autonomous Bodies (`ine.mx`), 32 States, and \~2,460 Municipalities. This fragmentation creates:

  * **Dark Corners:** "Zombie" websites and shadow infrastructure where accountability vanishes.
  * **Silos:** Agencies that should cooperate but share no digital connection.
  * **Opaque Procurement:** Bid-rigging rings that leave structural traces in hyperlink networks but are invisible to standard text search.

## 🔭 The Solution

NexoGob is a **"Digital MRI"** for the state. It crawls every accessible public URL, extracts entities (people, contracts, agencies), and constructs a massive **Knowledge Graph**.

### Key Features

  * **🌌 Galaxy View:** A GPU-accelerated visualization of millions of government nodes, color-coded by branch (Executive, Legislative, Judicial).
  * **🕵️‍♀️ Forensic Drill-Down:** Query the graph to find "cliques" of suppliers and agencies that form closed loops (a common red flag for bid-rigging).
  * **🚧 The Pathfinder:** Calculate the "bureaucratic friction" (click distance) for a citizen attempting to access a specific service.
  * **🧟 Zombie Detection:** Automatically flag domains that have been abandoned but still host sensitive public data.

-----

## 🏗️ Architecture

NexoGob uses a hybrid crawling strategy to handle both legacy server infrastructure and modern Single Page Applications (SPAs).

### 1\. The Spider (Ingestion)

  * **Seed Generator:** Auto-generates targets using the **INAI Padrón de Sujetos Obligados** and the **ADELA API**.
  * **Apache Nutch 🐘:** Handles the "Broad Crawl" of static content (millions of HTML/PDF documents).
  * **Crawlee (Node.js) 🎭:** A headless browser cluster for extracting data from dynamic portals (e.g., *DataMéxico*, *PNT*).
  * **Politeness Enforcement:** Uses **Kubernetes APF** concepts to throttle requests, preventing inadvertent DoS attacks on fragile municipal servers.

### 2\. The Brain (Storage & Logic)

  * **Neo4j 🕸️:** The core graph database. Stores the topology of the government.
  * **Elasticsearch:** Stores full-text content for semantic search.
  * **Graph Data Science Lib:** Runs nightly batch jobs for **PageRank** (centrality) and **Louvain Modularity** (community detection).

### 3\. The Lens (Visualization)

  * **Cosmograph 🪐:** WebGL-based engine to render 1M+ nodes in the browser without lag.
  * **Sigma.js:** Handles interactive sub-graph exploration.

-----

## ⚡ Getting Started (planned — not yet runnable)

### Prerequisites

  * Docker & Docker Compose
  * Node.js v18+
  * Java 11+ (for Nutch)
  * 8GB+ RAM (16GB recommended for full graph processing)

### Installation

1.  **Clone the repository**

    ```bash
    git clone https://github.com/your-org/nexogob.git
    cd nexogob
    ```

2.  **Spin up the infrastructure**
    This starts Neo4j, Elasticsearch, and the Nutch server.

    ```bash
    docker-compose up -d
    ```

3.  **Seed the Crawler**
    Run the discovery script to fetch the latest list of federal agencies from the ADELA API.

    ```bash
    npm run seed:federal
    ```

4.  **Start a Pilot Crawl**
    Target a specific sector (e.g., Health) to avoid overwhelming your machine.

    ```bash
    npm run crawl -- --sector=salud --depth=3
    ```

5.  **Launch the Dashboard**

    ```bash
    cd frontend && npm start
    ```

    Visit `http://localhost:3000` to view the graph.

-----

## 📐 Data Dictionary (Graph Schema)

### Nodes

| Label | Description | Properties |
| :--- | :--- | :--- |
| `Agency` | A legal government entity | `name`, `budget_id`, `state` |
| `Domain` | A root URL (e.g., `ine.mx`) | `ip_address`, `host_provider` |
| `Page` | A specific URL path | `title`, `http_status`, `hash` |
| `Contract` | A procurement record | `amount`, `date`, `currency` |
| `Person` | A public official | `name`, `position`, `salary_grade` |

### Edges

| Type | Description |
| :--- | :--- |
| `:LINKS_TO` | A hyperlink from Page A to Page B. |
| `:OWNS` | Agency ownership of a Domain. |
| `:MENTIONS` | A page text mentioning a specific Person or Company. |
| `:AWARDED` | Agency -\> Contract relationship. |

-----

## ⚖️ Legal & Ethical Disclaimer

**Read before scraping:**

1.  **Transparency vs. Privacy:** While the *Ley General de Transparencia* (LGTAIP) grants access to public information, the *Ley General de Protección de Datos Personales* (LGPDPPSO) strictly protects personal data.
2.  **PII Redaction:** NexoGob includes a regex-based **PII Scrubber** middleware. Do **NOT** disable this. It automatically redacts CURPs, private emails, and home addresses found in scraped documents.
3.  **Rate Limiting:** Default politeness policies are set to `Crawl-Delay: 5s`. Aggressive crawling of government servers may be interpreted as a cyberattack under the *Código Penal Federal*. **Use responsibly.**

-----

## 🗺️ Roadmap

  * [ ] **Phase 1 (Atlas):** Core crawling of `gob.mx` and Neo4j setup. *(designed, not yet implemented — see Current Status)*
  * [ ] **Phase 2 (Deep Dive):** Integration of State and Municipal domains + OCR for PDF scanning.
  * [ ] **Phase 3 (The Analyst):** Implementation of "Bid-Rigging Detection" algorithms (Cycle detection).
  * [ ] **Phase 4 (Public Access):** Release of the public-facing API and "State of the Digital Union" report.

## 🤝 Contributing

We welcome contributions from data scientists, frontend wizards, and policy experts. Please read `CONTRIBUTING.md` for details on our code of conduct and the process for submitting pull requests.

## 📄 License

This project is licensed under the MIT License - see the `LICENSE` file for details.

-----

*Built with 🇲🇽 by the NexoGob Team.*
