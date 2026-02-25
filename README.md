# Social Media Bot

The motivation for this project was to play around with up-and-coming generative AI technologies and explore how they could be used to fully automate social media for brands. Originally designed to run 24/7 on a Raspberry Pi, the bot operates autonomously but includes a human-in-the-loop mechanism, proactively requesting authorization for generated posts via Discord before publishing them.

**Main Features:**
- **Automated Content Generation**: Utilizes LLMs and custom generators to create scheduled, varied content like philosophical quotes, thoughts, and images.
- **Human-in-the-loop Approval**: Automatically integrates with Discord to let users review, approve, or reject generated content before it goes live.
- **Retrieval-Augmented Generation (RAG)**: Employs a local database and vector storage to ground generated texts in specific source materials.
- **Scheduling**: Relies on `APScheduler` to configure dynamic cron schedules and manage content queues effectively.
- **Multi-Platform Integration**: Includes modules for posting and scraping. Notably, the X/Twitter component employs browser automation to scrape pages as a workaround for the constraints of the paid API.
- **Social Graph Optimization (Experimental)**: Explored building out a social graph of connections to better understand user relationships and generate highly contextualized dynamic interactions with followers.

## Installation

### 1. Install Dependencies

> In some cases, ```numpy``` will fail to load the c-extensions when running on a Raspberry pi. To fix this, run
> ```sudo apt-get install libatlas-base-dev```.

First, set up the virtual environment using `uv`. Make sure you are in the ```social-media-bot``` directory:

```uv venv```

```source .venv/bin/activate```

> To deactivate the virtual environment, run ```deactivate```.

Then, install the requirements for the project:

```uv pip install -r requirements.txt```

### 2. Set up the systemd service

Now, set up the systemd service. To do this, modify line 7 and 8 in ```etc/social-media-bot.service``` to indicate the path 
of the ```src``` directory and ```main.py```, respectively. Then, copy the service file to ```/etc/systemd/system/``` and enable the service:

```sudo cp etc/social-media.service /etc/systemd/system/```

```sudo chmod +x main.py```

```sudo systemctl daemon-reload```

```sudo systemctl enable social-media.service```

```sudo systemctl start social-media.service```

Finally, verify the service is running:

```sudo systemctl status social-media.service```

## Architecture

The main logic resides in the `src/` directory.

### Scheduler (`src/scheduler.py`)
The core orchestrator based on APScheduler. Its main actions are:
1. Updating the local database by generating missing content.
2. Loading authorized content into the scheduler to be posted at their scheduled times.

### Generators (`src/generators.py`)
Contains functions that build the content (e.g., `image_with_quote`, `random_thought`). You can configure them or add new ones by defining new methods that return the required attributes.

### Content Objects (`src/content.py`)
Defines data structures (like `TwitterContentObject`) that bundle a generation function, an authorization function, a post function, and a cron schedule into a single manageable unit.

### Configuration (`src/config.yaml`)
The control center where you define the active scheduled tasks. You configure each task by specifying the target generator, authorization and post functions, along with its cron schedule and API keys path.

### Modules (`src/modules/`)
Helper modules providing specific functionalities:
- `discord_api.py`: Integrates with Discord, primarily used to request human authorization for generated content before posting.
- `twitter_api.py` & `twitter_web/`: Handle posting and scraping on X/Twitter.
- `openai_api.py` & `prompts.py`: Handle LLM requests and prompt templates.
- `sqlite_db.py` & `vector_db/`: Manage data storage and Retrieval-Augmented Generation (RAG) capabilities.
- `image_editor/`: Tools for manipulating and generating images for posts.
