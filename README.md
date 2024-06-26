# Learning Platform Tasks

This repository contains the structure and examples for creating tasks for a templating system to handle various types of learning tasks. In this structure we differ between templates and tasks while every task belongs to exactly one template and for each template there are multiple tasks that are using it. 

A template defines the overall layout and logic of a certain kind of task, e.g. what kind of input and output elements are present and how they are structured. A task itself then fills this template with specific content to create an individual exercise. 

Both templates and tasks contain code blocks that create the html elements, fill it with content and handle 
their behavior. In the examples below the scripts are written in python and are executed with pyscript. 
However this can just as well be implemented in plain JavaScript or other languages or libraries that can
access DOM elements.  

## Templates
### Definition: 
The template should build the overall layout of the task and defines which elements are present and how the user can interact with them. A template contains the following fields:

```yaml
description: A short description of the type of exercise and the contained elements
pyscript:
  imports: imports for all necessary python packages 
  generator: |
      def generate_template(parent):
        # python code that builds the html elements that should be displayed in the task
```

The generator function is called by the generator function from a task that uses this template. It gets a reference to the parten DOM element the task should rendered into.
It creates the input and output elments and define their layout and behavior. In the end the function
returns a reference to each element back to the task's generator function where they are filled with content.

### Example: 
This template defines a task that have a text-based question/task and a simple input field. 
Both elements are just empty container which are filled with content by the task itself. 

```yaml
description: Simple question field on the top and and answer field at the bottom
pyscript:
  imports: |
    from pyscript import document
    from pyweb import pydom
  generator: |
      def generate_template(parent):
        parent = document.getElementById(parent)
        parent.innerHTML=""

        question_element = document.createElement("div")
        answer_element = document.createElement("div")
        parent.append(question_element)
        parent.append(answer_element)

        return question_element, answer_element
```

## Tasks
### Definition
A task is always connected to a specific template and uses its elements to display a task.
It contains a title and description to describe what content will be visible and what is expected from the user.
It also contains a task-related script that picks or generates the specific exercise. For example the task could be to multiply two numbers and the numbers and the according solution themself are randomly generated following certain rules and restrictions. These task details can be accessed by application that uses the task. The task also provide a script that evaluate the user input and return wheter the given answer is right or wrong. Every text that is displayed is provided in different languages and the parent application is responsible for picking the according language. 

```yaml
title:
  german: german title 
  english: english title 
description: A detailed description of the task 
example: An example of how the exercise can look like
topic_id: Assign the task to a topic to structure and cluster tasks within a learning curriculum
template: the id of the template that is used 
text: Definition of multi-lingual text variables that can be referenced in the generator script
pyscript:
  imports: imports for all necessary python packages 
  globals: variables that should be gloablly accessable e.g. within the evaluation script 
  checker: |
    def check_answer(event):
        # this function evaluates the user input and returns wheter it is correct or not
        return [is_correct, answer, user_answer]
  generator: |
    def generate(parent):
      # calls the generator function to get the template elements. Then it generates the 
      # task details, e.g. the specific exercise that has to be solved and the according solution.
      # Then it manipulate the template elements to show the exercise 

  details: |
    def get_details():
      # Returns a text that contains all the generated task details to be used in the parent application
      # For example these details could be used as context for an AI-Tutor that assists the user to solve
      # the exercise. 

```

### Example: 
This task implements a exercise in which the user has to multiply two numbers which are multiples of ten.
It uses the the template the shows a question element and a single input element that was defined above. 

```yaml
title:
  german: Mit Zehnern multiplizieren
  english: Multiplying by tens
description: The student sees a multiplication exercise where both numbers are multiples
  of 10. The student have to finde the solution of the multiplication.
example: 'Exercise: 60*70 Correct Answer: 4200'
topic_id: 17
template: 01_simple_question_answer
text:
  text_question:
    german: 'Multipliziere.'
    english: 'Multiply.'
pyscript:
  imports: |
    import random
    from pyscript import document
    from pyweb import pydom
    from js import document
  
  globals: |
    answer = 0
    n1 = 0
    n2 = 0

  checker: |
    def check_answer(event):
      # Access the math-field element
      math_field = document.getElementById("equation")
      user_answer = math_field.getPromptValue('answer')
    
      if user_answer.isnumeric():
          user_answer = int(user_answer)
          is_correct = True if user_answer == answer else False
      else:
          is_correct = False

      return [is_correct, answer, user_answer]
  generator: |
    def generate(parent):
      global answer, n1, n2

      n1 = random.randint(1, 9) * 10
      n2 = random.randint(1, 9) * 10

      answer = n1 * n2
      equation = str(n1) + "\\cdot" +str(n2) +" = \\placeholder[answer]{}"

      question_element, answer_element = generate_template(parent)
      question_element.innerText="{{ text.text_question }}"
      answer_field = document.createElement("math-field")
      answer_field.id = "equation"
      answer_field.className = 'w-48'
      answer_field.setValue(equation)
      answer_field.mathVirtualKeyboardPolicy = "manual"
      answer_field.readonly = True
      answer_element.append(answer_field)

  details: |
    def get_details():
      return f"The current exercise is {n1}*{n2}. Therefore the correct answer is {answer}"
```

The task details that are generated in the generate function are the two numbers that has to be mutliplied and the according correct answer. These details are globally available to be used in the evaluator as well as in the get_details() function.  