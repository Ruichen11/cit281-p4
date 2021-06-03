# cit281-p4

Purpose of this project:
- Creating an REST API that works with a data source of questions and answers. 
- Gain experience interpreting functional descriptions and specification to complete an assignment 
- Gain experience using Fastify with the GET verb, routes, and route parameters
- Gain experience testing code module without using a web server
- Gain experience using Postman to test web server routes
- Gain experience working with JSON


## Project Code:
1. P4-data

// Question and answer data array
```
const data = [
  {
    question: "Q1",
    answer: "A1",
  },
  {
    question: "Q2",
    answer: "A2",
  },
  {
    question: "Q3",
    answer: "A3",
  },
];
```

// Export statement must be below data declaration - no hoisting with const
```
module.exports = {
  data,
};
```

```
function pluck(array, key) {
  return array.map((item) => item[key]);
}
```
2. getQuestions():
- Returns an array of strings where each array element is a question.
```
function getQuestions() {
  const questions = [];
  for (const qa of data) {
    questions.push(qa.question);
  }
  return questions;
```

3. getAnswers():
- Returns an array of strings where each array element is an answer.
```
function getAnswers() {
  return pluck(data, "answer");
}
```

4. getQuestionsAnswers():
- Returns a copy of the original data array of objects.
```
function getQuestionsAnswers() {
 return data.map((item) => {
    return { ...item };
  });
}
```

5. getQuestion(number = ""):

Returns an object with the following properties:

 * question property (string): The question from the data.
 
 * number property (integer): The question number,  not array index value.
 
 * error message property (string): Any error that occurred while getting the question.

```
function getQuestion(number = "") {
  // Prepare default response object
  const response = {
    error: "",
    question: "",
    number: "",
  };

  // Validate question number
  if (!Number.isInteger(number)) {
    response.error = "Question number must be an integer";
  } else if (number < 1) {
    response.error = "Question number must be >= 1";
  } else if (number > data.length) {
    response.error = `Question number must be less than the number of questions (${data.length})`;
  } else {
    // Get question for response
    index = number - 1;
    response.number = number;
    response.question = data[index].question;
  }

  return response;
}
```

6. getAnswer(number = ""):

 Returns an object with the following properties:

* answer property (string): The answer from the data.

* number property (integer): The question number,  not array index value.

* error message property (string): Any error that occurred while getting the question.

```
function getAnswer(number = "") {
  // Prepare default response object
  const response = {
    error: "",
    answer: "",
    number: "",
  };

  // Validate answer number
  if (!Number.isInteger(number)) {
    response.error = "Answer number must be an integer";
  } else if (number < 1) {
    response.error = "Answer number must be >= 1";
  } else if (number > data.length) {
    response.error = `Answer number must be less than the number of questions (${data.length})`;
  } else {
    // Get answer for response
    index = number - 1;
    response.number = number;
    response.answer = data[index].answer;
  }

  return response;
}
```

7. getQuestionAnswer(number = ""):

Returns an object with the following properties:

* question property (string): The question from the data.

* answer property (string): The answer from the data.

* number property (integer): The question number,  not array index value.

* error message property (string): Any error that occurred while getting the question.

```
function getQuestionAnswer(number = "") {
  // Get question and answer objects
  const question = getQuestion(number);
  const answer = getAnswer(number);

  // Validate question/answer objects
  if (question.error.length !== 0) {
    // Question error
    return question;
  } else if (answer.error.length !== 0) {
    // Answer error
    return answer;
  } else {
    // Both valid, combine into a response object
    return { ...question, answer: answer.answer };
  }
}
```

- Import data
```
// Import initial static data
const { data } = require("./p4-data.js");

// Export functions - hoisted
module.exports = {
  getQuestion,
  getAnswer,
  getQuestionAnswer,
  getQuestions,
  getAnswers,
  getQuestionsAnswers,
  };
 ```
 
### Fastify Server Code: 
```
const fastify = require("fastify")();
const {
  getQuestion,
  getAnswer,
  getQuestions,
  getAnswers,
  getQuestionsAnswers,
  addQuestionAnswer,      // Extra credit
  updateQuestionAnswer,   // Extra credit
  deleteQuestionAnswer    // Extra credit
} = require("./p4-module.js");
```
```
fastify.get("/cit/question", (request, reply) => {
  // Return response
  reply
    .code(200)
    .header("Content-Type", "text/json; charset=utf-8")
    .send({ error: "", statusCode: 200, questions: getQuestions() });
});
```

```
fastify.get("/cit/question/:number", (request, reply) => {
  // Extract question number using deconstruction
  let { number = "" } = request.params;

  // Setup default response object
  const response = {
    error: "",
    statusCode: 200,
    question: "",
    number: "",
  };

  // Process request
  if (number === "") {
    // Question number was not specified
    response.error = "Number route parameter required";
    response.statusCode = 404;
  } else {
    // Valid route parameter, convert to number
    number = parseInt(number);

    // Get question
    const questionInfo = getQuestion(number);

    // Check response
    if (questionInfo.error.length > 0) {
      // Error getting question
      response.error = questionInfo.error;
      response.statusCode = 404;
    } else {
      // Valid question found and returned
      response.question = questionInfo.question;
      response.number = number;
    }
  }

  // Return response
  reply
    .code(response.statusCode)
    .header("Content-Type", "text/json; charset=utf-8")
    .send(response);
});
```
```
fastify.get("/cit/answer", (request, reply) => {
  // Return response
  reply
    .code(200)
    .header("Content-Type", "text/json; charset=utf-8")
    .send({ error: "", statusCode: 200, answers: getAnswers() });
});
```
```
fastify.get("/cit/answer/:number", (request, reply) => {
  // Extract answer number using deconstruction
  let { number = "" } = request.params;

  // Setup default response object
  const response = {
    error: "",
    statusCode: 200,
    answer: "",
    number: "",
  };

  // Process request
  if (number === "") {
    // Answer number was not specified
    response.error = "Number route parameter required";
    response.statusCode = 404;
  } else {
    // Valid route parameter, convert to number
    number = parseInt(number);

    // Get answer
    const answerInfo = getAnswer(number);

    // Check response
    if (answerInfo.error.length > 0) {
      // Error getting answer
      response.error = answerInfo.error;
      response.statusCode = 404;
    } else {
      // Valid answer found and returned
      response.answer = answerInfo.answer;
      response.number = number;
    }
  }

  // Return response
  reply
    .code(response.statusCode)
    .header("Content-Type", "text/json; charset=utf-8")
    .send(response);
});
```

```
fastify.get("/cit/questionanswer/:number", (request, reply) => {
  // Extract question/answer number using deconstruction
  let { number = "" } = request.params;

  // Setup default response object
  const response = {
    error: "",
    statusCode: 200,
    question: "",
    answer: "",
    number: "",
  };

  // Process request
  if (number === "") {
    // Answer number was not specified
    response.error = "Number route parameter required";
    response.statusCode = 404;
  } else {
    // Valid route parameter, convert to number
    number = parseInt(number);

    // Get answer
    const answerInfo = getAnswer(number);

    // Check response
    if (answerInfo.error.length > 0) {
      // Error getting answer
      response.error = answerInfo.error;
      response.statusCode = 404;
    } else {
      // Valid answer found and returned
      response.answer = answerInfo.answer;
      response.number = number;
    }
  }

  // Return response
  reply
    .code(response.statusCode)
    .header("Content-Type", "text/json; charset=utf-8")
    .send(response);
});
```

```
fastify.get("/cit/questionanswer", (request, reply) => {
  // Return response
  reply.code(200).header("Content-Type", "text/json; charset=utf-8").send({
    error: "",
    statusCode: 200,
    questions_answers: getQuestionsAnswers(),
  });
});
```

```
// Handle unmatched routes
fastify.get("*", (request, reply) => {
  reply
    .code(404)
    .header("Content-Type", "text/json; charset=utf-8")
    .send({ error: "Route not found", statusCode: 404 });
});
```

### What I learned:
- Gain more experience with coding along with working with fastify. additional, worked with Node.js REST API to handle GET verbs for data. 
