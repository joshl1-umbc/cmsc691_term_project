# Dice Roll Action Generating Operational Network

Welcome to the _Dice Roll Action Generating Operational Network (DRAGON)._
The project was completed by Shawn Bray, Josh Li, and Reece Robertson in fulfillment of the course requirements for Dr. Lara Martin's _CMSC 691—Interactive Fiction and Text Generation_ at the University of Maryland, Baltimore County.
A complete demonstration video for this project can be found at: [https://umbc.webex.com/umbc/ldr.php?RCID=3fa1cc86aa42eb0f334fae3eef62663a](https://umbc.webex.com/umbc/ldr.php?RCID=3fa1cc86aa42eb0f334fae3eef62663a).

## Introduction

DRAGON is a system which generates action effect descriptions in response to player input.
The output of the model is evaluated using a quantum cost function, along with the traditional BLEU and ROGUE metrics.

This document outlines the requirements and setup required to run DRAGON, and then provides instructions for gameplay and a complete example.
Following this is a discussion of the **LAMBEQ Score**, a novel method of natural language sentence similarity evaluation using the [Quantinuum Lambeq](https://cqcl.github.io/lambeq-docs/) Python package.
The LAMBEQ score leverages quantum states for its comparison, and represents an exciting foray into quantum natural language processing.
Finally, this document concludes with a project status statement.

### LLM Use Statement
We did not use an LLM to generate any part of our code files, our report file, or this document.
We do use the Llama 2 LLM to generate response descriptions to player actions.

## Requirements

This project utilizes one of Meta's [Llama 2](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf) large language models.
In order to access this model, one wishing to run our system must complete the following steps:
1. Create a [Hugging Face](https://huggingface.co/) account.
2. In your [token settings](https://huggingface.co/settings/tokens), generate an access token with write permissions. 

## Setup Instructions

For best results we recommend uploading a copy of `final_project.ipynb` to [Google Colab](https://colab.research.google.com/).
Then select `Runtime->Run All` to execute all cells of the notebook.
In the first cell you will be propted to input the Hugging Face access token you generated in the second step of the **Requirements** section.
Following this you will be prompted to `Add token as a git credential? (Y/n)`,to which you can reply with `n`.
That completed, scroll down to the end of the notebook to find the section entitled **Run the Game**.
Wait for this cell to execute, and then enjoy!

## Usage Instructions

Once the final cell of the notebook begins to execute, gameplay begins.
Gameplay in DRAGON consists of a series of turns.
First, the DRAGON will query Meta's Llama 2 model to generate some text.
On the first turn, this will be a brief description of the fantasy world in which the player now finds themself.
On subsequent turns, this will be a brief description of the effect of the prior turn's action.

Following DRAGON's prompt, the player may then type a natural language sentence to describe their desired action.
DRAGON will augment this text description with a randomly generated dice roll result, and prompt the Llama 2 model to generate a description for the resulting action.
The model is instructed to generate a two-part output: first, there should be a six-word sentence describing the success or failure value of the action, and second, there should follow a 2-3 sentence natural language description of the action result.
The first sentence is then compared to a success/failure sentence hypothesis given the dice result.
This comparison is made using the [BLEU](https://www.nltk.org/api/nltk.translate.bleu_score.html) score, the [ROGUE](https://pypi.org/project/rouge-score/) score, and a new evaluation metric called the LAMBEQ score, which is described below in the **LAMBEQ Score** section.
The validation values of all three metrics are printed to the screen, and then a new turn begins.
Turns continue indefinitely until the player instructs the model to `quit`.
The following section presents an example of gameplay.

## Example Gameplay

> Welcome to the AI Generated Adventure!
> 
> Your situation: You are creating an interactive fiction game. Describe the initial setting and situation the player finds themselves in. Keep it to 2-3 sentences.
> 
> The player finds themselves in a dense forest, surrounded by tall trees that block out most of the sunlight. The air is cool and damp, and the only sounds are the distant chirping of birds and the rustling of leaves in the wind. As they look around, they notice that the forest is eerily quiet, and they can't shake the feeling that they are being watched.
> 
> What would you like to do? (type 'quit' to end)
> 
> \> I light a torch and throw it into a clearing to draw attention away from me.
> 
> \[DICE ROLL: 8\]
> 
> Your torch illuminates a small clearing, and you notice a figure hiding behind a nearby tree. The figure is holding a crude weapon, and it looks like they are preparing to attack.
> 
> Target:		 Your torch illuminates a small clearing,
> 
> Hypothesis:	 Your action was a slight failure.
> 
> BLEU:		   0.6875
>
> ROUGE:		 0.3333333333333333
>
> LAMBEQ:		 0.9871439344010786
>
> What would you like to do? (type 'quit' to end)
> 
> \> I open a portal to the ethereal dimension and shove the attacker through it.
> 
> \[DICE ROLL: 19\]
> 
> Your action was a critical success. The attacker is sucked into the ethereal dimension, leaving you momentarily vulnerable but relieved that the immediate threat has been neutralized. However, you know that the attacker may still be able to return to the material plane through other means, and you must remain vigilant to prevent any further attacks. Your familiar, who had been watching the scene unfold, now approaches you and informs you that it has discovered a hidden entrance to a secret underground network. The entrance is located deep within the nearby forest, and your familiar offers to lead you to it in exchange for a favor.
>
> Target:		 Your action was a critical success.
>
> Hypothesis:	 Your action was a major success.
>
> BLEU:		   0.8375
>
> ROUGE:		 0.8333333333333334
>
> LAMBEQ:		 0.9751956246432688
> 
> What would you like to do? (type 'quit' to end)
>
> \> quit
> 
> Thanks for playing!

## LAMBEQ Score

The _LAMBEQ score_ is a new similarity evaluation metric introduced by the DRAGON system.
As mentioned above, this metric is built using Quantinuum Lambeq, which is a quantum natural language processing package for Python.
It also utilizes [Quantinuum Tket](https://docs.quantinuum.com/tket/), a Python package for quantum circuit representation and evaluation, and [IBM Qiskit](https://www.ibm.com/quantum/qiskit), a similar tool to Tket, used here to execute quantum circuits on a simulated hardware backend.

The LAMBEQ score is evaluated using the following steps:
1. Two natural language sentences are input into the LAMBEQ score function.
2. These sentences are decomposed into DisCoCat grammatical string diagrams using Lambeq's `sentence2diagram` function.
3. These string diagrams are compiled into quantum circuit objects using a Lambeq quantum ansatz.
4. All parameterized gates in each circuit are populated with a constant value, which completes the embedding of the natural language sentences into a quantum representation.
5. The expectation value of each circuit is computing (we chose to perform this computation using an IBM Quantum `AerBackend` simulator and a Z-basis measurement observable, but any quantum device and measurement observable can be used).
6. The distance between the expectation values is computed using a similarity function, which outputs a real number between 0 and 1 that represents the relative similarity of the two input sentences.

## Project Status

This project is completed.
We have no intent of continuing this work, accepting pull requests, or accepting other contributions.
