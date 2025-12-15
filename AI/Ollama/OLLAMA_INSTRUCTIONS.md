Based on your successful installation, you are currently sitting at the active prompt (`>>>`) for the model.



Here is the isolated set of instructions to manage this model effectively, specifically focusing on performance (keepalive) and resource management for your \*\*RTX 5070 Ti\*\*.



\### 1\\. Exit the Current Session



Right now, you are in a standard session with a default 5-minute timeout. To apply the optimized settings, you need to exit first.



  \* \*\*Command:\*\* Type `/bye` and hit Enter.



\### 2\\. The "Power User" Start Command (Keepalive)



This is the primary command you should use to start the model when you plan to code for a few hours. It forces the model to stay loaded in your VRAM, ensuring every subsequent prompt gets an \*\*instant\*\* response without reloading.



  \* \*\*Command:\*\*

    ```powershell

    ollama run qwen2.5-coder:14b --keepalive 2h

    ```

    \*(Note: You can change `2h` to `30m` (30 minutes) or `24h` depending on your needs. `2h` is usually the sweet spot.)\*



\### 3\\. Verify It Is Loaded in VRAM



Since you have 16GB of VRAM and this model takes up about 9GB, you want to ensure it isn't spilling over into your slower system RAM.



  \* \*\*Open a NEW PowerShell window\*\* (leave the chat window open) and run:

    ```powershell

    ollama ps

    ```

  \* \*\*What to look for:\*\*

      \* \*\*Processor:\*\* Should say `100% GPU`.

      \* \*\*Until:\*\* This column tells you when the model will auto-unload (e.g., "2 hours from now").



\### 4\\. How to Unload (Free up GPU for Gaming)



If you decide to play a game or do heavy rendering, you need to kick the model out of your VRAM immediately, or your game performance will tank.



  \* \*\*Command:\*\*

    ```powershell

    ollama stop qwen2.5-coder:14b

    ```



\### 5\\. Utilizing Your Context Window



With your hardware profile and this specific 14B model, you have a distinct advantage: \*\*Massive Context\*\*.



  \* \*\*The Math:\*\* The model uses \\~9GB. You have 16GB total. This leaves \\~7GB free.

  \* \*\*The Capability:\*\* You can paste roughly \*\*20,000 to 30,000 tokens\*\* of code into the prompt.

  \* \*\*Usage:\*\* You don't need to feed it snippets. You can paste an entire documentation file or a large class file and ask, \*"Refactor this entire file to use Pattern X."\*



\### Summary Checklist



1\.  \*\*Start work:\*\* `ollama run qwen2.5-coder:14b --keepalive 2h`

2\.  \*\*Check status:\*\* `ollama ps`

3\.  \*\*End work:\*\* `ollama stop qwen2.5-coder:14b`

