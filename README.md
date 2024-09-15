```markdown
# Text to Math Problem Solver and Data Search Assistant

This Streamlit app uses LangChain and Groq's Gemma model to provide an interactive interface for solving math problems and searching for information. It integrates various tools and models to handle user queries effectively.

## Features

- **Math Problem Solver**: Answer mathematical questions with detailed explanations.
- **Wikipedia Search**: Retrieve information on various topics from Wikipedia.
- **Logic and Reasoning**: Solve logic-based and reasoning questions.

## Setup

### Prerequisites

Ensure you have the following installed:

- Python 3.10 or higher
- Streamlit
- LangChain and LangChain-Groq libraries

### Installation

1. Clone the repository:

   ```bash
   git clone <repository_url>
   cd <repository_directory>
   ```

2. Create and activate a virtual environment:

   ```bash
   python -m venv env
   source env/bin/activate  # On Windows use `env\Scripts\activate`
   
      Windows using conda
   conda create -p venv python==3.10 -y
   conda activate venv
    
   ```

3. Install the required packages:

   ```bash
   pip install -r requirements.txt
   ```

4. Add your Groq API key in a `.env` file or directly in the code.

### Running the App

1. Start the Streamlit app:

   ```bash
   streamlit run app.py
   ```

2. Open the provided local URL in your browser to interact with the app.

## Code Explanation

### Imports and Setup

```python
import streamlit as st
from langchain_groq import ChatGroq
from langchain.chains import LLMMathChain, LLMChain
from langchain.prompts import PromptTemplate
from langchain_community.utilities import WikipediaAPIWrapper
from langchain.agents.agent_types import AgentType
from langchain.agents import Tool, initialize_agent
from langchain.callbacks import StreamlitCallbackHandler
```

- **`streamlit`**: Creates the web app interface.
- **`ChatGroq`**: Interacts with the Groq API for language models.
- **`LLMMathChain`**: Processes math queries.
- **`LLMChain`**: Integrates language model with prompt template.
- **`PromptTemplate`**: Defines the prompt format.
- **`WikipediaAPIWrapper`**: Searches Wikipedia for information.
- **`Tool` and `initialize_agent`**: Set up tools and agents.
- **`StreamlitCallbackHandler`**: Manages Streamlit callbacks.

### Streamlit App Setup

```python
st.set_page_config(page_title="Text To Math Problem Solver And Data Search Assistant", page_icon="🧮")
st.title("Text To Math Problem Solver Using Google Gemma 2")

groq_api_key = st.sidebar.text_input(label="Groq API Key", type="password")

if not groq_api_key:
    st.info("Please add your Groq API Key to continue")
    st.stop()

llm = ChatGroq(model="Gemma2-9b-It", groq_api_key=groq_api_key)
```

- Sets the title and icon for the app.
- Provides an input field for the Groq API key.
- Initializes the language model.

### Tool Initialization

```python
wikipedia_wrapper = WikipediaAPIWrapper()
wikipedia_tool = Tool(
    name="Wikipedia",
    func=wikipedia_wrapper.run,
    description="A tool for searching the Internet to find various informations on the topics mentioned"
)

math_chain = LLMMathChain.from_llm(llm=llm)
calculator = Tool(
    name="Calculator",
    func=math_chain.run,
    description="A tool for answering math-related questions. Only input mathematical expression need to be provided"
)
```

- Initializes tools for Wikipedia searches and math problem solving.

### Prompt and Chain Setup

```python
prompt = """
You are an agent tasked with solving users' mathematical questions. Logically arrive at the solution and provide a detailed explanation
and display it point-wise for the question below
Question:{question}
Answer:
"""

prompt_template = PromptTemplate(
    input_variables=["question"],
    template=prompt
)

chain = LLMChain(llm=llm, prompt=prompt_template)

reasoning_tool = Tool(
    name="Reasoning tool",
    func=chain.run,
    description="A tool for answering logic-based and reasoning questions."
)
```

- Defines how prompts should be structured and sets up chains for processing questions.

### Agent Initialization

```python
assistant_agent = initialize_agent(
    tools=[wikipedia_tool, calculator, reasoning_tool],
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=False,
    handle_parsing_errors=True
)
```

- Initializes the agent with tools for handling various types of queries.

### Streamlit Interaction

```python
if "messages" not in st.session_state:
    st.session_state["messages"] = [
        {"role": "assistant", "content": "Hi, I'm a math chatbot who can answer all your math questions"}
    ]

for msg in st.session_state.messages:
    st.chat_message(msg["role"]).write(msg['content'])

question = st.text_area("Enter your question:", "I have 5 bananas and 7 grapes. I eat 2 bananas and give away 3 grapes. Then I buy a dozen apples and 2 packs of blueberries. Each pack of blueberries contains 25 berries. How many total pieces of fruit do I have at the end?")

if st.button("Find my answer"): 
    if question:
        with st.spinner("Generating response.."):
            st.session_state.messages.append({"role": "user", "content": question})
            st.chat_message("user").write(question)

            st_cb = StreamlitCallbackHandler(st.container(), expand_new_thoughts=False)
            response = assistant_agent.run(st.session_state.messages, callbacks=[st_cb])
            st.session_state.messages.append({'role': 'assistant', "content": response})
            st.write('### Response:')
            st.success(response)
    else:
        st.warning("Please enter the question")
```

- Manages the chat interface, processes user questions, and displays responses.

## Troubleshooting

- Ensure the Groq API key is correctly set.
- Verify that all dependencies are installed and compatible.
- Check the Streamlit logs for any runtime errors.

## Contributing

Feel free to submit issues or pull requests to improve the app or fix bugs.

## License

This project is licensed under the MIT License.

```
