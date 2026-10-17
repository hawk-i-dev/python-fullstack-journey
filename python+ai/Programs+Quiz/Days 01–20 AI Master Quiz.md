# Days 1–20 AI Master Quiz

This covers Python foundations, data handling, RAG, LLMs, agents, safety, tracing, and evaluations. An agent system combines instructions, tools, memory/state, orchestration, safety, and quality checks. [OpenAI building-agents guide](https://developers.openai.com/tracks/building-agents)

Reply in this format:

```text
1.B
2.A
3.C
...
30.D
```

1. What is the main purpose of type hints in Python?

   A. They always stop invalid values at runtime  
   B. They improve code clarity, editor support, and static checking  
   C. They remove the need for tests  
   D. They automatically convert values  

2. Why should a function validate its inputs?

   A. To handle invalid data early and predictably  
   B. To make all functions longer  
   C. To avoid returning values  
   D. To remove error handling  

3. Which structure stores named key-value pairs?

   A. `list`  
   B. `tuple`  
   C. `dict`  
   D. `set`  

4. What is the safest way to handle an expected runtime error?

   A. Ignore it  
   B. Use `try/except`, handle it clearly, and log useful details  
   C. Delete the code  
   D. Restart the computer  

5. Why are unit tests important?

   A. They prove the UI is beautiful  
   B. They verify small pieces of logic work as expected  
   C. They replace Git  
   D. They remove all production bugs forever  

6. What is a regression test?

   A. A test for a failure that previously happened  
   B. A test that always fails  
   C. A test that requires an API key  
   D. A test only for CSS  

7. Why use `default_factory=list` in a dataclass?

   A. To share the same list across all objects  
   B. To create a fresh list for every object  
   C. To prevent lists in Python  
   D. To make the list immutable  

8. Why save application state as JSON?

   A. It is readable and easy to exchange between systems  
   B. It automatically encrypts secrets  
   C. It removes the need for validation  
   D. It replaces databases in every case  

9. What is the goal of clean project structure?

   A. Put all code into one file  
   B. Separate responsibilities so code is easier to change and test  
   C. Avoid using modules  
   D. Remove documentation  

10. What does Git help you do?

   A. Track changes and collaborate safely  
   B. Train an LLM  
   C. Replace Python  
   D. Store API keys publicly  

11. What is vector similarity used for in a basic RAG system?

   A. Deleting documents  
   B. Finding text chunks semantically related to a query  
   C. Creating frontend pages  
   D. Sending emails  

12. Why do we chunk long documents before retrieval?

   A. To make files larger  
   B. To retrieve smaller, more focused pieces of relevant information  
   C. To remove all metadata  
   D. To avoid embeddings  

13. What is the correct RAG flow?

   A. Ask model → invent answer → retrieve later  
   B. Clean → chunk → retrieve → provide context → answer with sources  
   C. Delete documents → call model  
   D. Only store complete PDFs in prompts  

14. If retrieval finds no relevant context, what should the assistant do?

   A. Confidently invent an answer  
   B. Use unrelated chunks  
   C. Say that it could not find supporting information  
   D. Delete the knowledge base  

15. What is the key difference between a normal chatbot and an agent?

   A. Agents can use controlled tools to retrieve data or perform actions  
   B. Agents never use Python  
   C. Chatbots always use databases  
   D. Agents never need instructions  

16. In tool calling, who executes a custom function?

   A. The LLM directly  
   B. Your Python application  
   C. The JSON schema  
   D. The user’s browser automatically  

17. Why must your Python code validate tool-call arguments?

   A. Tool arguments may be invalid, incomplete, or unsafe  
   B. The model cannot produce JSON  
   C. Validation is only for frontend code  
   D. Python never raises errors  

18. What is a tool allowlist?

   A. A list of tools the agent is explicitly permitted to use  
   B. A list of all possible internet commands  
   C. A list of API keys  
   D. A type of database  

19. What is the safest response to an unknown tool request?

   A. Execute it immediately  
   B. Deny it safely  
   C. Give it admin access  
   D. Delete all logs  

20. What does an agent plan do?

   A. Lists the controlled steps needed to fulfill a request  
   B. Stores user passwords  
   C. Replaces validation  
   D. Removes the need for tools  

21. When should an agent ask a clarification question?

   A. When required details are missing  
   B. After every successful request  
   C. Only when a test passes  
   D. Never  

22. When is a multi-agent handoff useful?

   A. When a specialist should take ownership of that conversation branch  
   B. For every small function  
   C. To skip security checks  
   D. To avoid documentation  

23. What is a guardrail?

   A. An automatic safety or validation check  
   B. A UI component  
   C. A database table  
   D. An API key  

24. Which action normally requires explicit approval?

   A. Searching local documents  
   B. Calculating a budget  
   C. Deleting an expense  
   D. Categorizing an expense  

25. Why should approval tokens expire and be one-time use?

   A. To prevent old or leaked approvals from being reused  
   B. To make them easier to guess  
   C. To avoid audit logs  
   D. To remove user identity checks  

26. Which data should never be written directly to audit logs?

   A. Tool name  
   B. Timestamp  
   C. API key or password  
   D. Approval status  

27. What is a trace?

   A. A record of one workflow run and its meaningful steps  
   B. A Python virtual environment  
   C. A database password  
   D. A Git branch  

28. What is an evaluation dataset used for?

   A. Repeatedly checking expected system behavior across test cases  
   B. Replacing user authentication  
   C. Avoiding debugging  
   D. Storing API keys  

29. What is the best improvement cycle for an AI system?

   A. Deploy immediately and ignore failures  
   B. Inspect trace → identify cause → add eval → fix → rerun tests  
   C. Delete logs after every run  
   D. Change many files randomly  

30. What is the strongest overall rule from Days 1–20?

   A. Trust every model response  
   B. Build quickly without tests  
   C. Give agents unrestricted tool access  
   D. Build AI features that are validated, safe, tested, observable, and improvable
