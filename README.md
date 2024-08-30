# Text Correction Application with LangGraph and LangChain<a href="https://medium.com/@givkashi/text-correction-application-with-langchain-langgraph-and-llm-bbd0ad006160" target="blank"><img align="center" src="https://raw.githubusercontent.com/rahuldkjain/github-profile-readme-generator/master/src/images/icons/Social/medium.svg" alt="@_giaabaoo_" height="30" width="40" /></a>


## Overview

This repository contains the implementation of a text correction application using LangChain and LangGraph libraries. The application is designed to identify grammatical mistakes, rewrite sentences for clarity, and enhance the overall quality of the text. It leverages the power of a Large Language Model (LLM), specifically the **meta-llama/llama-3.1-8b-instruct** model, to provide accurate and context-aware text corrections.

<center>
    <a href="https://github.com/givkashi/Text-Correction-Application-with-LangGraph/blob/main/LangGraph_Text_Correction.ipynb"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>
</center>

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Workflow Explanation](#workflow-explanation)
  - [1. Grammatical Error Detection](#1-grammatical-error-detection)
  - [2. Text Correction and Improvement](#2-text-correction-and-improvement)
  - [3. Output Compilation](#3-output-compilation)


## Introduction

This project demonstrates how to build a text correction application by integrating LangChain with LangGraph. The application uses a graph-based workflow to process text, identify and correct errors, and provide an enhanced version of the original input. The notebook is designed to be easily extendable and can be adapted to different models or additional processing steps.

## Features

- **Automated Text Correction:** Identifies and corrects grammatical errors in text inputs.
- **Text Improvement:** Enhances the quality of text by rewriting sentences for clarity and coherence.
- **Graph-Based Workflow:** Uses LangGraph to create a modular and extensible workflow for text processing.
- **Streaming Output:** Supports streaming output to visualize the step-by-step corrections.

## Installation

To run this application, you need to install the required libraries. You can do this by running the following command:

```bash
pip install langchain langchain_openai langgraph
```

Ensure that your environment is configured with Python 3.7+.

## Usage

After installing the dependencies, you can run the text correction application by executing the following script:

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import Graph

# Set the model as ChatOpenAI
model = ChatOpenAI(model='meta-llama/llama-3.1-8b-instruct', temperature=0)

# Define functions for each node in the workflow
def function_1(state):
    messages = state['messages']
    user_input = messages[-1]
    complete_query = """You are an English writer assistant. Find all the grammatical mistakes in the provided text by the user and write the correct version with explanation for each one
            \n Here is the user text: """ + user_input
    response = model.invoke(complete_query)
    state['messages'].append(response.content)
    return state

def function_2(state):
    messages = state['messages']
    agent_response = messages[-1]
    complete_query = """Rewrite the text and correct all the mistakes also improve the quality of the text """ + agent_response
    response = model.invoke(complete_query)
    state['messages'].append(response.content)
    return state

def function_3(state):
    messages = state['messages']
    user_input = messages[0]
    first_response = messages[1]
    second_response = messages[-1]
    output =  user_input + "\n " + first_response + "\n " + second_response
    return output

# Build the workflow using LangGraph
workflow = Graph()
workflow.add_node("agent1", function_1)
workflow.add_node("agent2", function_2)
workflow.add_node("responder", function_3)
workflow.add_edge('agent1', 'agent2')
workflow.add_edge('agent2', 'responder')
workflow.set_entry_point("agent1")
workflow.set_finish_point("responder")

# Compile the application
app = workflow.compile()

# Provide input for correction
inputs = {"messages": ["Helo am fine wat are you doin. Nice too met you"]}
app.invoke(inputs)

# Stream the output
input = {"messages": ["Helo am fine wat are you doin. Nice too met you"]}
for output in app.stream(input):
    for key, value in output.items():
        print(f"Output from node '{key}':")
        print("---")
        print(value)
    print("\n---\n")
```

## Workflow Explanation

The application workflow is built using LangGraph and consists of three main stages:

### 1. Grammatical Error Detection

- **Function:** `function_1`
- **Description:** This function identifies grammatical mistakes in the input text and provides a corrected version along with explanations for each correction.

### 2. Text Correction and Improvement

- **Function:** `function_2`
- **Description:** After identifying the errors, this function rewrites the text, correcting all mistakes and improving the overall quality for better readability.

### 3. Output Compilation

- **Function:** `function_3`
- **Description:** This function compiles the original text, the identified errors and corrections, and the final improved text into a single output.

