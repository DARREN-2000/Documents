**Tools Research**

**1. Grafana and Prometheus**

**Approach:**
The benchmark script would push metrics to Prometheus (a time-series database). Grafana connects to Prometheus and displays real-time dashboards. Both run as K8s services.

**Setup:**

1. Install Prometheus using Helm chart
2. Install Grafana using Helm chart
3. Write a small exporter that converts our JSON to Prometheus metrics
4. Design dashboard templates in Grafana UI

**Manual work after setup:**
Basically none. Run benchmark, metrics automatically flow to Prometheus, dashboards update in real-time.

**License:** Apache 2.0 ✓

**Honest take:** This is solid if you're already running Prometheus somewhere. The exporter to convert JSON to metrics is maybe an hour's work. But if you're starting fresh, setting up Prometheus and Grafana feels heavier than what we actually need. You're getting a sledgehammer when you need a screwdriver. That said, if metrics monitoring is already part of your infrastructure, this integrates beautifully.

------------------------------------------------------------------------

**2. MLflow**

**What it is:**
Experiment tracking platform from the ML community. Originally built for managing machine learning experiments.

**Approach:**
Add a few lines to our benchmark script to log results. MLflow stores everything and provides a web UI to view and compare runs.

**Setup:**

1. Deploy MLflow server in K8s (Helm chart available)
2. Set up PostgreSQL backend for persistence
3. Add logging calls to benchmark script

**Manual work after setup:**
None. The logging happens automatically when we run benchmarks.

**License:** Apache 2.0 ✓

**Honest take:** This one actually works well. It's purpose-built for what we're doing. The learning curve is gentle, and you get experiment versioning and comparison built in. We tested this and it's solid. The UI is a bit dated, but it gets the job done. PostgreSQL backend might feel like overkill for smaller teams, but it's reliable and performant. Worth trying.

------------------------------------------------------------------------

**3. Apache Superset**

**What it is:**
Full-featured business intelligence platform, basically the open-source version of Tableau.

**Approach:**
Load JSON results into a PostgreSQL database. Use Superset's web interface to build rich, interactive dashboards with various chart types.

**Setup:**

1. Deploy PostgreSQL in K8s
2. Deploy Superset (requires Redis for caching)
3. Write Python script to parse JSON and insert into database
4. Create dashboards using Superset's visual interface

**Manual work after setup:**
After each benchmark, run the data loading script. Dashboards automatically reflect new data.

**License:** Apache 2.0 ✓

**Honest take:** Superset is powerful but feels overkill for benchmark tracking. You're getting SQL querying capabilities, advanced visualization types, and team collaboration features that we probably won't use. Setup is more involved—you need PostgreSQL, Redis, and the data loading pipeline. The UI builder is excellent, but for just displaying benchmark results, you're using maybe 20% of what Superset offers. Good if you already have databases everywhere and want a unified BI platform across the company. Otherwise, it's extra complexity we don't need.

------------------------------------------------------------------------

**4. Metabase**

**What it is:**
Business intelligence tool focused on simplicity. Easier to use than Superset, more intuitive for people who aren't technical.

**Approach:**
Similar to Superset—load JSON into database, but create dashboards using a much more intuitive drag-and-drop interface. No SQL knowledge required.

**Setup:**

1. Deploy PostgreSQL
2. Deploy Metabase (simpler than Superset, single pod)
3. Load benchmark data into database
4. Build dashboards through point-and-click interface

**Manual work after setup:**
Upload new results to database after benchmarks. Dashboards refresh automatically.

**License:** AGPL v3

**Skipping because:** AGPL v3 requires that any modifications we make must be open-sourced. If we modify Metabase or integrate it heavily with our code, we'd have to release those changes publicly. That's a deal-breaker for our internal tools. If you're fine with that license requirement, Metabase is actually very nice to use.

------------------------------------------------------------------------

**5. Streamlit**

**What it is:**
Python framework that turns scripts into interactive web applications. Think of it as a rapid way to turn your Python code into a shareable dashboard.

**Approach:**
Write a Python script that reads our JSON files and creates visualizations. Streamlit converts it into a web interface with interactive elements. Deploy as a simple web service in K8s.

**Setup:**

1. Write dashboard.py with our visualization logic
2. Create Docker container
3. Deploy to K8s as a web service

**Manual work after setup:**
Just drop new JSON files in the results folder. Dashboard automatically picks them up.

**License:** Apache 2.0 ✓

**Honest take:** This might actually be the best fit for quick iteration. We get exactly what we need, nothing more. The whole thing feels lightweight. Since most of our team knows Python, building and updating the dashboard is fast. The downside is it doesn't have built-in experiment versioning like MLflow does. But honestly, if your goal is just to visualize results and share them with stakeholders quickly, this is hard to beat. We could prototype everything in a couple hours.

------------------------------------------------------------------------

**6. Redash**

**What it is:**
Open-source dashboarding tool focused on SQL queries and sharing results.

**Approach:**
Load JSON into database, write SQL queries in Redash, create visualizations from query results.

**Setup:**

1. Deploy using Helm chart
2. Set up PostgreSQL database
3. Load benchmark data
4. Create queries and build dashboards

**Manual work after setup:**
Load new results into database. Dashboards update automatically.

**License:** BSD 2-Clause ✓

**Honest take:** Redash is good if you're comfortable with SQL. The UI is clean and dashboards share nicely. But it's another database to manage and requires writing SQL queries every time you want a new visualization. For benchmark results that don't change structure often, this is probably more work than Streamlit. It's like choosing between "write a Python script" versus "write SQL queries"—depends on your team's strength. If everyone knows SQL and wants to query data ad-hoc, Redash makes sense.

------------------------------------------------------------------------

**7. Jupyter Notebooks + Voilà**

**What it is:**
Using Jupyter for analysis, then Voilà to convert notebooks into standalone web applications. Basically lets you take a notebook and make it look like a real web app.

**Approach:**
Create a Jupyter notebook that loads and visualizes our benchmark results. Deploy with Voilà to make it accessible as a web page (hides the code from users).

**Setup:**

1. Create notebook with analysis and visualizations
2. Deploy Voilà server in K8s
3. Configure to auto-execute notebook

**Manual work after setup:**
Notebook runs automatically when accessed. Very low maintenance.

**License:** BSD 3-Clause ✓

**Honest take:** This is a middle ground between custom code and pre-built tools. If your team is already in Jupyter, it's convenient. But Voilà has performance issues when too many people access it simultaneously. Also, notebooks can become messy and hard to maintain as they grow. We'd probably end up rewriting it as a proper app anyway. Good for small proof-of-concepts, but I wouldn't bet on it for production tracking.

------------------------------------------------------------------------

**8. Plotly Dash**

**What it is:**
Python framework for building analytical web applications. Similar to Streamlit but more focused on complex, production-grade dashboards.

**Approach:**
Write Python code using Dash components to build an interactive web dashboard. Deploy as a web service in K8s.

**Setup:**

1. Develop dashboard using Dash (similar to Streamlit but more code)
2. Containerize the application
3. Deploy to K8s

**Manual work after setup:**
Drop JSON files in folder. Dashboard reads them automatically.

**License:** MIT ✓

**Honest take:** Plotly Dash is more powerful than Streamlit but requires writing more code. If we need advanced interactive features and don't mind extra complexity, Dash is solid. It's used in production at major companies. But for just displaying benchmark results, it feels like we'd be building more than we need. Choose Dash over Streamlit if you know you'll need complex interactions and want production-grade stability.

------------------------------------------------------------------------

**9. Panel (part of HoloViz)**

**What it is:**
Python library for creating custom dashboards and apps. Works standalone or inside Jupyter. Sits between Streamlit's simplicity and Dash's complexity.

**Approach:**
Write Python code to create interactive dashboards from our JSON files. Deploy as a web app in K8s.

**Setup:**

1. Develop dashboard using Panel
2. Deploy as web app in K8s

**Manual work after setup:**
Minimal—reads JSON files automatically.

**License:** BSD 3-Clause ✓

**Honest take:** Panel is less popular than Streamlit but actually quite flexible. It's less opinionated than Streamlit, so you have more control. Performance is generally good. The problem is community size—fewer Stack Overflow answers when you get stuck. It's not bad, it's just less mainstream than Streamlit or Dash. If you like tinkering and don't mind smaller communities, Panel is worth trying.

------------------------------------------------------------------------

**10. Weights & Biases (W&B)**

**What it is:**
Industry-standard experiment tracking platform. Designed specifically for ML workflows. The platform everyone talks about at ML conferences.

**Approach:**
Integrate W&B SDK into benchmark scripts. Automatically logs metrics, parameters, and saves images. Web UI provides experiment versioning, comparison, and team collaboration.

**Setup:**

1. For cloud: Create account, add API key to scripts, done
2. For self-hosted: Deploy using Helm operator, configure storage backend
3. Add wandb.init() and wandb.log() calls to benchmark code
4. Images upload automatically

**Manual work after setup:**
None. Everything logs automatically during benchmark runs. Real-time dashboards update as data arrives.

**License:** MIT (SDK is open), but server is proprietary

**Skipping because:** W&B's self-hosted version isn't free. The server license starts around $8K/year for smaller teams. Cloud version has free tier available (limited), but if you want self-hosted with support, budget for it. We're exploring open-source alternatives that don't have licensing costs. Cloud tier works fine for small teams though—worth trying their free tier first.

------------------------------------------------------------------------

**11. Kubeflow**

**What it is:**
End-to-end ML platform for Kubernetes. Includes pipelines, experiment tracking, notebook servers, and model serving. The heavy-duty option.

**Approach:**
Deploy Kubeflow on K8s cluster. Convert benchmark scripts into pipeline components. Results stored as artifacts, visualized in Kubeflow UI.

**Setup:**

1. Deploy Kubeflow (full installation on K8s)
2. Create pipeline YAML for benchmark execution
3. Configure artifact storage
4. Set up TensorBoard for visualizations

**Manual work after setup:**
Moderate. Initial setup is involved. Converting scripts to pipeline components takes work. But once running, pipelines execute automatically.

**License:** Apache 2.0 ✓

**Honest take:** Kubeflow is powerful but honestly feels like overkill for just tracking benchmark results. You get a complete ML platform with orchestration, hyperparameter tuning, and model serving. But if we only need to store and visualize benchmark data, we're not using 80% of what Kubeflow offers. The learning curve is steep. Better choice if you're already using K8s heavily and want everything unified. For just benchmarks, choose something simpler.

------------------------------------------------------------------------

**12. ClearML**

**What it is:**
MLOps platform similar to W&B but fully open-source. Includes experiment tracking, orchestration, and artifact storage. Good middle ground between lightweight tools and heavy platforms.

**Approach:**
Add ClearML Task to benchmark scripts. Automatically captures environment, code changes, and results. Deploy ClearML server to K8s with MongoDB and Elasticsearch backends.

**Setup:**

1. Deploy ClearML server using Helm (requires MongoDB, Elasticsearch)
2. Configure artifact storage (S3 or local)
3. Initialize Task in benchmark code
4. Log metrics and upload images

**Manual work after setup:**
Minimal. ClearML auto-detects code changes and captures environment info. Images and JSON artifacts are versioned automatically.

**License:** Apache 2.0 ✓

**Honest take:** ClearML is like what W&B would be if it was fully open-source. The feature set is comprehensive. The UI is capable but feels a bit cluttered compared to W&B. Infrastructure setup is heavier—you need MongoDB and Elasticsearch running. Once running though, it's very reliable. If you need full control and don't want licensing costs, ClearML is genuinely worth the setup effort. It's what we'd move to if W&B licensing becomes an issue.

------------------------------------------------------------------------

**13. Neptune.ai**

**What it is:**
Experiment tracking focused on metadata management. Strong for organizing large numbers of runs. Designed for teams that run hundreds of experiments and need good organization.

**Approach:**
Initialize Neptune runs in benchmark code. Log metrics, parameters, and files. Cloud or self-hosted deployment with web UI for browsing results.

**Setup:**

1. For cloud: API key setup, done
2. For self-hosted: Deploy using installer, configure ClickHouse/MySQL/Redis
3. Add Neptune.init_run() to code
4. Log results and files

**Manual work after setup:**
None after integration. All logging automatic during benchmark execution.

**License:** Proprietary (self-hosted requires commercial license)

**Skipping because:** Self-hosted Neptune needs a commercial license. Cloud tier is available but ties you to their service. We want something we own and control. Neptune is excellent for large-scale experiments (1000+ runs), but we're not at that scale yet. Revisit if we scale up and need advanced metadata management.

------------------------------------------------------------------------

**14. Aim**

**What it is:**
Lightweight, open-source experiment tracking. Purpose-built to handle 10,000+ runs efficiently without UI slowdown.

**Approach:**
Add Aim tracking to benchmark code. Deploy Aim server to K8s. Remote tracking sends data to server, UI stays responsive even with huge datasets.

**Setup:**

1. Deploy Aim server to K8s (uses persistent volume for storage)
2. Add Aim Run tracking to benchmark code with remote URL
3. Access UI to visualize results

**Manual work after setup:**
None. Real-time logging. UI updates as benchmarks run.

**License:** Apache 2.0 ✓

**Honest take:** Aim is refreshing. It's designed for exactly what we're doing—fast, lightweight tracking. The UI is snappy even with massive datasets. Setup is simpler than ClearML or Neptune. The downside is smaller community—fewer integrations and fewer Stack Overflow answers. But if you don't need those extras, Aim actually works better than heavier tools because it's built for speed. Worth trying if lightweight and fast is what you care about.

------------------------------------------------------------------------

**15. DVC (Data Version Control)**

**What it is:**
Version control for ML workflows and data. Like Git, but for your entire ML experiment pipeline.

**Approach:**
Use DVC to version benchmark scripts, configurations, and results. Commit to Git, store artifacts in S3/MinIO. Creates a reproducible experiment history.

**Setup:**

1. Install DVC
2. Configure remote storage (S3/MinIO/local)
3. Add benchmark pipeline to DVC
4. Commit results with Git

**Manual work after setup:**
Quite a bit. DVC requires discipline—you need to properly structure pipelines and commit results regularly. It's not automatic.

**License:** Apache 2.0 ✓

**Honestly not using it because:** DVC is great for data versioning and reproducibility, but it's not designed for dashboarding or real-time visualization. If we adopt DVC, we'd still need another tool (MLflow, W&B, Streamlit, etc.) to actually visualize results. It's an addition to your workflow, not a replacement for tracking and visualization.

------------------------------------------------------------------------

**16. Pachyderm**

**What it is:**
Data versioning platform similar to DVC but more enterprise-focused. Reproducible data pipelines with full lineage tracking.

**Approach:**
Define benchmark pipeline in YAML. Pachyderm tracks data lineage and versioning. Integrates with K8s native.

**Setup:**

1. Deploy Pachyderm to K8s
2. Define pipeline specs
3. Data flows through versioned stages

**Manual work after setup:**
Initial pipeline setup is technical. After that, runs automatically.

**License:** Apache 2.0 (Community Edition)

**Honestly not using it because:** Pachyderm is powerful for reproducible pipelines, but again, it's not for visualization. Also, licensing complexity—enterprise features require payment. And setup is fairly involved if you're new to it. For just benchmarks, this feels overengineered. Use DVC or Pachyderm if reproducibility and data lineage are critical requirements beyond just visualization.

------------------------------------------------------------------------

**17. Tableau**

**What it is:**
Industry-leading business intelligence platform. The gold standard for dashboarding and visualization.

**Approach:**
Connect to data source containing benchmark results. Build dashboards using Tableau's UI.

**Setup:**

1. Deploy Tableau Server to K8s (or use cloud)
2. Connect to database with benchmark results
3. Build dashboards

**Manual work after setup:**
Build dashboards once. They update automatically when data refreshes.

**License:** Proprietary (expensive)

**Skipping because:** Tableau licensing is steep—starting around $2K/month for a server license. And K8s deployment is complex. For a team just tracking benchmarks, the cost and complexity aren't justified. If your company already has Tableau licenses and infrastructure, sure, use it. Otherwise, it's overkill financially.

------------------------------------------------------------------------

**18. Sisense**

**What it is:**
Another enterprise BI tool, similar positioning to Tableau.

**Approach:**
Connect benchmark database, build dashboards through UI.

**Setup:**

1. Deploy to K8s
2. Connect to data
3. Create dashboards

**Manual work after setup:**
Dashboard building then automatic updates.

**License:** Proprietary (enterprise pricing)

**Skipping because:** Similar to Tableau—enterprise pricing, complex deployment, overkill for benchmarks. If you're already an enterprise customer, it's fine. Otherwise, stick with open-source options.

------------------------------------------------------------------------

**19. Databricks**

**What it is:**
Managed Spark platform with notebooks and dashboarding. Combines data processing and visualization.

**Approach:**
Upload benchmark data to Databricks. Run SQL queries and create dashboards in notebooks.

**Setup:**

1. Set up Databricks workspace
2. Upload data
3. Build dashboard notebooks

**Manual work after setup:**
Moderate. Need to maintain notebooks and data pipelines.

**License:** Proprietary (subscription)

**Skipping because:** Databricks is great if you're doing heavy data processing, but for just visualizing benchmark results, you don't need it. Plus, subscription cost and vendor lock-in. Only makes sense if your team is already using Databricks for other things.

------------------------------------------------------------------------

**20. W&B Self-hosted**

**What it is:**
Self-hosted version of W&B that runs entirely on your infrastructure.

**Approach:**
Deploy W&B server to K8s, integrate SDK into benchmark code.

**Setup:**

1. Deploy W&B server using K8s manifests
2. Configure S3/object storage for artifacts
3. Add wandb.init() to code
4. Access UI

**Manual work after setup:**
None. Same as cloud W&B.

**License:** Proprietary (commercial license required)

**Skipping because:** Self-hosted W&B needs a license from day one. If we're going to pay for self-hosting, ClearML or Aim provide similar functionality without license costs. Only worth it if W&B ecosystem is critical to your workflow.

------------------------------------------------------------------------

## Tools We're NOT Using - The Reality Check

**Why we're skipping Tableau, Sisense, Databricks:**
Licensing costs are significant ($2K-10K+/year). For benchmark tracking alone, the ROI doesn't justify it. These tools make sense if you're already paying for them or if you need them for other use cases.

**Why we're skipping DVC and Pachyderm:**
These handle versioning and reproducibility, but they don't give you visualization dashboards. You'd still need another tool. If reproducibility isn't critical, they add complexity without solving the visualization problem.

**Why we're not all-in on Kubeflow:**
It's powerful but feels like enterprise software for a problem we can solve with something lighter. If we're already building a full ML platform, sure, use it. But just for benchmarks? Too much overhead.

**Why Metabase and Neptune licensing is a blocker:**
AGPL and proprietary licenses either open-source our code or cost money. Open-source alternatives exist that don't have these constraints.

**Why K8s-native isn't always better:**
Kubeflow being "native to K8s" is nice in theory, but it doesn't mean simpler for our specific use case. ClearML and Aim also run on K8s without requiring a complete platform overhaul.

------------------------------------------------------------------------

## What We're Actually Considering - The Short List

**Top 3:**
1. **MLflow** - Balance of features and simplicity
2. **Streamlit** - If we want lightweight and fast iteration
3. **ClearML** - If we need full features and own everything

**Honorable mention:**
- **Aim** - Excellent if speed is critical
- **Grafana + Prometheus** - If already in your infrastructure

**What we're testing first:**
We're going with MLflow for production-ready experiment tracking and prototyping a Streamlit dashboard simultaneously to see which feels right for stakeholder reporting.