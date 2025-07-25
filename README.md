# DevLake Automation Script

## Purpose

This script automates:
- Scanning all configured plugins and connections in DevLake.
- Syncing repositories from remote sources (GitHub, GitLab, Bitbucket, Jira, Azure) into DevLake.
- Creating projects and scope configs for each repository.
- Running data collection for each project, with retry and alert logic for failures.


## Key Features

- **Automatic plugin and connection discovery**: No manual configuration required.
- **Repository sync from remote**: Automatically adds new repos from GitHub/GitLab/... to the connection.
- **Automatic project and blueprint creation**: Each repo gets its own project and blueprint.
- **Batch data collection**: Collects data for each blueprint in batches, retries on failure, and alerts if stuck.
- **Can run standalone, via Docker, or as a Kubernetes Job.**

## Usage

### 1. Run locally (Python 3.9+ required)

```bash
pip install -r requirements.txt
export API_BASE_URL=http://localhost:4000  # or your DevLake backend endpoint
python main.py
```

### 2. Run with Docker

```bash
docker build -t devlake-script .
docker run --rm -e API_BASE_URL=http://localhost:4000 devlake-script
```

### 3. Run on Kubernetes (see job.yaml)

- Edit `API_BASE_URL` in `job.yaml` as needed.
- Build & push the image to your registry, update the `image:` field in the yaml.
- Apply the job:
```bash
kubectl apply -f job.yaml
```

## Possible Improvements

- **Logging**: Currently uses print statements; should switch to Python logging for log files and log levels. Integrate with notification systems (e.g., Slack, email, or monitoring tools) to alert SRE teams on errors, stuck jobs, or important events.
- **Configurable batch size, retry, timeout**: Allow configuration via environment variables or command-line arguments.
- **Better error handling**: Refactor retry and alert logic into separate functions. Improve robustness and error recovery, especially when running with more than 200 repositories (e.g., handle API rate limits, timeouts, and partial failures gracefully).


## Code Structure

- `main.py`: The main entry point. Orchestrates the workflow by calling high-level functions to create projects/blueprints, update blueprint time windows, and trigger batch data collection. Keeps the main logic clean and delegates all business logic to modules.
- `modules/`:
  - `tools_plugins.py`: Contains functions for interacting with DevLake plugins and connections. Handles plugin discovery, connection listing, scope (repo) retrieval, and remote group/repo fetching. All API calls related to plugin/connection metadata are here.
  - `tools_projects.py`: Contains logic for creating projects, blueprints, and scope configs in DevLake. Handles syncing repositories from remote sources, creating or updating project/blueprint records, and managing scope configuration for each repo. Also includes utility functions for time calculations.
  - `tools_blueprints.py`: Implements batch data collection logic. Handles triggering blueprint data collection, monitoring pipeline status, retrying failed jobs. Contains all logic for orchestrating and monitoring data collection batches.

---

