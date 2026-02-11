Minigame Framework Overview
This project implements a modular minigame system for Roblox, designed to manage and execute a sequence of interactive challenges through a centralized manager.
The architecture utilizes a FIFO queue with integrated shuffling logic to provide a randomized gameplay experience triggered by player interaction. 
The framework currently supports four minigames: a math-based arithmetic challenge, a logic-oriented Simon Says variant featuring trustworthy and untrustworthy prompts,
a rhythm-based timing game utilizing tweening mechanics, and a typing accuracy test with real-time RichText feedback. Each module is designed to be self-contained,
handling its own UI lifecycle, input connections, and cleanup routines to ensure a scalable and performant implementation for any game environment.
