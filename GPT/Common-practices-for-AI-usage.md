# Common practices for AI usage

This guide was created for startup companies that don't have a budget to spend on corporate accounts yet.

**Tables of Contents**

- [Common practices for AI usage](#common-practices-for-ai-usage)
  - [Guidelines for Using on specific GPTs.](#guidelines-for-using-on-specific-gpts)
    - [Claude.ai](#claudeai)
    - [ChatGPT \& ChatGPT Plus](#chatgpt--chatgpt-plus)
    - [Github Copilot](#github-copilot)
    - [Gemini](#gemini)
    - [Perplexity.ai](#perplexityai)
    - [Codeium](#codeium)
  - [Guidelines for Using GPT in Working Tasks](#guidelines-for-using-gpt-in-working-tasks)

Guidelines for Using on specific GPTs.
--------------------------------------

### Claude.ai

- No need to do anythings. They don't use our data to train their model. [How do you use personal data in model training? | Anthropic Help Center](https://support.anthropic.com/en/articles/7996885-how-do-you-use-personal-data-in-model-training)

### ChatGPT & ChatGPT Plus

- Please opt-out for allow them to use data to train their models.
  - <https://help.openai.com/en/articles/7730893-data-controls-faq>
  - [How your data is used to improve model performance | OpenAI Help Center](https://help.openai.com/en/articles/5722486-how-your-data-is-used-to-improve-model-performance#h_6dea59578a)

### Github Copilot

- Please opt-out for allow Github to use data for improve their products.

- [Managing Copilot policies as an individual subscriber - GitHub Docs](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/managing-copilot-policies-as-an-individual-subscriber#enabling-or-disabling-prompt-and-suggestion-collection)

### Gemini

- No need to do anythings. They don't use our data to train their model. [How Gemini for Google Cloud uses your data](https://cloud.google.com/gemini/docs/discover/data-governance#submit-receive-data)

### Perplexity.ai

- Please toggle off for allow AI Data Usage. [What data does Perplexity collect about me?](https://www.perplexity.ai/hub/faq/what-data-does-perplexity-collect-about-me)

### Codeium

- Please opt-out for telemetry data. Go to Settings page then remove checkbox from "Disable code snippet telemetry". [Codeium - Free AI Code Completion & Chat](https://codeium.com/settings)

  - They declared in here that actually they don't use input or output data from any users even individual account. [How is Codeium Free?](https://codeium.com/blog/how-is-codeium-free)

  - They declared that only data that they gathered is telemetry data such as latency, engagement with features, and suggestion acceptance information that you can opt-out by above setting. [Security and Privacy | Codeium - Free AI Code Completion & Chat](https://codeium.com/security)

Guidelines for Using GPT in Working Tasks
-----------------------------------------

1.  **Confidentiality and Data Privacy**
    - **Rule**: Never input sensitive business information, proprietary code, or any confidential data into GPT.
    - **Explanation**: GPT interactions are not guaranteed to be private, and there is a risk that input data could be stored or accessed unintentionally by third parties.
    - **Example**: Instead of pasting actual code or client details, describe the issue in general terms, e.g., "I'm working on a user authentication system and encountering a logic issue with token generation."

2.  **Abstraction Over Specificity**
    - **Rule**: Use high-level abstractions when discussing business logic or proprietary processes.
    - **Explanation**: This prevents the accidental sharing of specific algorithms or business methods that are proprietary.
    - **Example**: Say "We need a function to handle complex data validation" instead of "We need a function to validate credit card information and store it in our database."

3.  **Role-Specific Usage Guidelines**

    - **Project Managers (PM)**:
      - **Rule**: Use GPT for general project management advice, timeline estimations, or communication templates.
      - **Example**: Ask "What are some best practices for managing a remote team?" instead of "How should I manage our confidential project with Company X?"

    - **Product Owners (PO)**:
      - **Rule**: Use GPT for market research, user feedback analysis, or high-level feature brainstorming.
      - **Example**: Ask "What are some trending features in mobile apps?" instead of "How should we implement our new price suggestion feature?"

    - **Software Engineers**:
      - **Rule**: Use GPT for learning new programming concepts, debugging strategies, or non-proprietary code snippets.
      - **Example**: Ask "How do I write a unit test in Python?" instead of "How do I debug this booking algorithm?"

4.  **Output Verification**
    - **Rule**: Always verify the output provided by GPT before integrating it into any project or documentation.
    - **Explanation**: GPT can generate plausible but incorrect information. Verification ensures accuracy and reliability.
    - **Example**: If GPT suggests a coding solution, test it thoroughly in a safe environment before deploying it in your production codebase.

5.  **Educational and General Research Use**
    - **Rule**: Use GPT for educational purposes and general research that does not involve sensitive or proprietary information.
    - **Explanation**: GPT is an excellent tool for expanding knowledge and understanding broader concepts without risking information leakage.
    - **Example**: Ask "What are the principles of Agile methodology?" instead of "How should we modify our specific Agile process for the new client?"

6.  **Ethical and Responsible Use**
    - **Rule**: Use GPT responsibly, ensuring it aligns with the company's values and ethical guidelines.
    - **Explanation**: GPT's outputs can sometimes be biased or inappropriate. It's crucial to use it in a way that promotes ethical standards.
    - **Example**: Avoid using GPT to generate content that could be considered offensive or discriminatory, even if it's for testing purposes.
