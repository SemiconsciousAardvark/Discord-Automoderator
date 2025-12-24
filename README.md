# Discord Political Automoderator 
### Licensed under AGPLv3

Version 1.0.0- fully functional but has not been tested outside of a custom server bot environment. Image moderation has not been tested beyond the first filter. Modlog settings are designed to integrate with YAGPDB.xyz and might not work without use of that bot. Most code is original, with Claude Code (Sonnet 4.5) used for organization and optimization.

This bot was developed as a side project to help manage Discord debate/politics communities and help eliminate moderator bias and inactivity.
It has a near- to above- human level accuracy in active and neutral chat moderation for the tested purposes (i.e. political environments).

It is made to strongly augment and assist, but not replace, human moderation. Unless its emergency mode is continuously running, it has a unique "style" to the messages it detects and acts on and may miss some flagrant violations. Even if its emergency mode is on, it may punish innocuous messages. Think of it as inviting small team of hyperactive volunteers who never get tired or stop monitoring your server; useful, but not final.

Prompts may be customized to different servers, but with unknown effects on performance. From testing, it is known that shorter, direct, and more objective prompts increase performance. However, the default prompts/policies took (and take?) a lot of feedback, time, and effort to get the wording right and more objective (LLMs have their own way of interpreting events). If your use case differs too much from neutral and strict political moderation with highly precise and objective policies, you will have to do significant experimentation for the bot to be helpful.

Lastly, since the bot was (and continues to be) developed for politics-friendly environments, the default policies defined in config/settings.py strictly prohibit some dark jokes and other banter that some communities find acceptable, while permitting other jokes and comments that some may find offensive.

```
automoderator/
├── config/              # Configuration and settings
│   ├── __init__.py
│   ├── clients.py       # Discord and API client initialization
│   └── settings.py      # All configurable constants and policies
├── moderation/          # Core moderation logic
│   ├── text_moderation.py    # Text content analysis
│   ├── image_moderation.py   # Image content analysis
│   └── toxicity_model.py     # ML toxicity classifiers
├── events/              # Discord event handlers
│   └── handlers.py      # Message, reaction, and member events
├── commands/            # Slash commands
│   └── slash_commands.py     # All bot commands
├── utils/               # Utility functions
│   ├── punishment.py    # Punishment system and modlog publishing
│   ├── leveling.py      # User leveling and heat tracking
│   ├── database.py      # SQLite database operations
│   └── message_utils.py # Message history and processing
├── manager.py           # Worker pool management
└── __main__.py          # Entry point
```

## APIs & Acknowledgments

- Huggingface models! specifically unitary/toxic-bert, Hate-speech-CNERG/bert-base-uncased-hatexplain, and openai/clip-vit-large-patch14
- Google Gemini API for advanced content analysis
- Google Perspective API for toxicity pre-filtering
- CatCord API for TOS violation checking

- Political Debate discord server for testing & development

# Quickstart

If you like and use this script, please include a link to this git in your bot's about me!

### Requirements
- Python 3.9 or higher
- A Discord bot token
- Google Gemini API key (paid tier strongly recommended for data privacy & Discord ToS compatibility)
- Google Perspective API key (optional but highly recommended for cost effectiveness)

### Cost Estimate (Paid Tiers)
I don't have good data to accurately estimate cost yet, but this is based on some daily testing. 
"Nice" environments will be cheaper, while more "toxic" environments will be more expensive

Standard Mode: ~$5-10 per 1 million messages

Emergency Mode: ~$50-100 per 1 million messages (assuming continuous emergency operation)

For comparison, a moderately active server of around 10k members usually sees around 300-400k messages per month

### Setup
- Add your server details and customization to config/settings.py
- Add a .env with your keys
- Install dependencies:
```bash
pip install -r requirements.txt
```
- Run the bot:
```bash
python3 -m automoderator
```

# Quicklook
(this section was generated mostly by Claude Code with some additions)

### Multi-Stage Moderation Pipeline

1. **Pre-filtering**: Local ML models (toxicity, hate speech) and PerspectiveAPI for fast initial screening
2. **First Pass**: Gemini analysis with confidence and severity scoring
3. **Review Stage**: Complex cases get clause-by-clause evaluation
4. **Validation**: Final check before punishment (optional)
5. **Emergency Mode**: Skip local and PerspectiveAPI classification and use Gemini to scan every message. This minimizes false negatives but is expensive

### Heat-Based Punishment System

Users accumulate "heat" across multiple time windows (weekly, biweekly, monthly, bimonthly). Punishment duration scales with all:
- Severity of the current violation
- User's heat scores
- Time-decay function for past offenses

Punishment time is determined by the user's heat score with an exponential fit (via Lagrange polynomials and smooth interpolation functions) to the pattern
 1 hour -> 1 day -> 7 days -> 14 days -> 28 days -> Indefinite mute

### Community Assisted Moderation (Flagging)

Users can flag messages with the 🚩 reaction. This bypasses any pre-filtering to avoid false negatives. The system includes:
- Per-user cooldowns to prevent spam
- Accuracy tracking (successful vs. unsuccessful flags)
- Automatic cooldowns for users with low accuracy (<60%)
- Rolling window for accuracy calculation
- Leaderboard and statistics commands

## Available Commands

### Moderation Commands
- `/analyze [message_link]` - Analyze a specific message (Manager only)
- `/modlogs [user_id]` - View a user's moderation history (Staff+)
- `/undo [user_id] [case_number] [reason]` - Remove a modlog entry (Deputy+)

### Settings Commands
- `/setting_text_punish [option]` - Toggle automatic punishments (Manager only)
- `/setting_misinfo [option]` - Toggle misinformation responses (Manager only)
- `/emergency_mode [option]` - Toggle fast classification mode (Admin only)

### Flagging System
- `/flag_stats [user_id]` - View flagging statistics
- `/flag_leaderboard` - View top flaggers by accuracy
- `/flag_analytics` - System-wide flagging analytics (Staff+)
- `/flag_reset_cooldown [user_id] [reason]` - Reset user cooldown (Manager only)
- `/flag_reset_stats [user_id] [reason]` - Reset user stats (Manager only)

### OSINT
- `/osint_lookup [user_id]` - Check TOS violations via CatCord API (Staff+)

## Customization

### Moderation Policies

Edit policies in `config/settings.py`:

- **text_policies**: Define text moderation rules
- **img_policies**: Define image moderation rules
- **rules_exact**: Precise policy definitions with clauses
- **rules_addenda_descriptive**: Guiding conditions for edge cases

### Punishment Tuning

Adjust punishment durations in `punishment_dict`:

```python
punishment_dict = {
    "INSULT": {
        0.5: 60,    # Low severity: 1 minute
        0.8: 300,   # Medium severity: 5 minutes
        1.0: 600    # High severity: 10 minutes
    },
    # ... more rules
}
```

### Sensitivity Thresholds

Tune Perspective API thresholds in `sensitivity_thresholds`:

```python
sensitivity_thresholds = {
    "TOXIC": 0.8,
    "INSULT": 0.6,
    "THREAT": 0.5,
    # ...
}
```

## Security Considerations

- All API keys are stored in `.env`
- Server-specific IDs are centralized in `config/settings.py`
- Image processing includes decompression bomb protection
- SQL injection prevention via parameterized queries
- Username sanitization to prevent prompt injection
- Rate limiting and cooldowns on user actions

## Database

Uses SQLite for persistent storage:

- **User profiles**: Heat scores, engagement indices, moderation flags
- **Moderation logs**: Detailed records of all punishments
- **Statistics**: Leveling, message counts, and toxicity scores

Database automatically migrates from legacy CSV files on first run.
