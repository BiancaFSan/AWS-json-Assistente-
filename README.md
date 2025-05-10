# AWS-json-Assistente-
Código do projeto criado em aula da DIO Criando um Assistente de Delivery com AWS Step Functions e Bedrock

{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Rio de Janeiro",
  "QueryLanguage": "JSONata",
  "States": {
    "Rio de Janeiro": {
      "Type": "Pass",
      "Next": "Ideias de restaurante",
      "Assign": {
        "counter": "{% $count($states.input.prompts) %}",
        "conversation_history": [
          ""
        ],
        "input_prompts": "{% $reverse($states.input.prompts) %}"
      }
    },
    "Ideias de restaurante": {
      "Type": "Choice",
      "Choices": [
        {
          "Next": "Ideias de comidas e bebidas da região",
          "Condition": "{% $counter > 0 %}"
        }
      ],
      "Default": "Success"
    },
    "Success": {
      "Type": "Succeed",
      "Output": "{% $join($conversation_history[[1..$count($conversation_history)]], '.') %}"
    },
    "Ideias de comidas e bebidas da região": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "cohere.command-text-v14",
        "Body": {
          "prompt": "{% $conversation_history[-1] & '.' & $input_prompts[$counter - 1] %}",
          "max_tokens": 250
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Assign": {
        "conversation_history": "{% $append($conversation_history, $states.result.Body.generations[0].text) %}",
        "counter": "{% $counter - 1 %}"
      },
      "Next": "Ideias de restaurante"
    }
  }
}
