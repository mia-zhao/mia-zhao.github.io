+++
title = 'A Taste in XState With React: Managing Logics With Finite State Machines'
date = 2024-03-14T00:00:00-00:00
draft = true
+++

## Introduction
Managing states in frontend applications can be a complex task, and it can lead to numerous challenges if not handled carefully. As developers, we frequently grapple with the decision of whether or not to use a state management library and if so, which one to choose. Popular choices for state management libraries include Redux (and its toolkit), MobX, Recoil, Zustand, and others. Each library aims to solve specific problems, and the choice really depends on your use case.

With the introduction of Server Components in React, straightforward CRUD (Create, Read, Update, Delete) applications may no longer need an additional state management library. However, for applications with heavy interactions and intricate logics, a dedicated state management solution may still be necessary.

## The Case Study: Masking and Unmasking Data with GPT
I recently started developing an application to wrap a GPT (Generative Pre-trained Transformer) model, specifically to mask sensitive data before sending it and unmask the data after receiving the results. The logic seemed relatively straightforward:

As the user types, a NER (Named Entity Recognition) model classifies entities such as names, cities and organizations. These classified entities are then masked based on the results. Upon clicking "Send", the masked message is sent to the OpenAI backend. After receiving the results from OpenAI, the entities are unmasked, and the entities are returned to the user.

Initially, React's `useReducer` hook might seem like a viable solution for managing the application state:


```
function messageReducer(message: Message, action: Action): Message {
  if (action.type === "encode") {
    const encoded = encode(action.text);
    return { ...message, encoded };
  } else if (action.type === "decode") {
    // ...
  } else {
    // ...
  }
}

function Component() {
  const [message, dispatch] = useReducer(messageReducer, initialMessage);
  return (
    <>
      <button onClick={() => dispatch({ type: "encode", text: "User Input" })}>
        Send
      </button>
    </>
  );
}
```

This approach appears clean, but revisiting the flow (encode message -> send to OpenAI -> receive message -> decode message) reveals a potential issue. According to the React documentation, the reducer

> must be pure, should take the state and action as arguments, and should return the next state.

Given this constraint, `useReducer` may not be the best approach for handling the entire flow, as it would require side effects (e.g., sending the message to OpenAI) within the reducer function, which goes against its intended purpose.

Another option to consider is using libraries such as Redux with Redux-Thunk for handling asynchronous actions. However, setting up with these libraries for this specific use case might be overkill, as it introduces additional complexity and boilerplate code.

## XState: Modeling Logic as a Finite State Machine
Considering the flow again: encode message -> send to OpenAI -> receive message -> decode message. This can be modeled as a finite state machine. XState, a state management library, might be a suitable solution for this use case. The states within the XState framework are declarative and the steps are well-defined.

Here's how we can model the states:

1. **Idle**: Initially, we would need an idle state for awaiting user input

2. **Encoded**: As the user types, the message should be encoded, thus the state transition from idle to encoded

3. **Query**: When the user clicks the "Send" button, we enter the query step. Two further notes:

* the query step performs an asynchronous function, and we need to handle both the success and failure cases

* the "Send" action can happen at any time, so the transition to the query state can occur from the idle state or the encoded state (or any subsequent states)

4. **Queried**: If the user action is successful, we enter the queried state; if not, we step back to the encoded state

5. **Decoded**: When in the queried state, the response is successfully grasped and we can transition seamlessly to the decoded state.

6. **Reset**: Immediately after the decoded, we perform a "reset" action, which potentially saves the current message data and clears the current states of the machine.

7. **Idle** (again): After the reset, we enter the idle for the next round of input. while reset state could simply be the idle state, we keep it separate for clearer flows.

Here's what the state machine would look like using XState:


```
const chatMachine = createMachine({
  id: "chat",
  initial: "Idle",
  context: initialContext,
  states: {
    Idle: {
      on: {
        encode: { /*...*/ },
        query: "Query",
      },
    },
    Encoded: {
      on: {
        encode: { /*...*/ },
        query: "Query",
      },
    },
    Query: {
      invoke: {
        src: "queryFunction",
        input: ({ event }) => {
          assertEvent(event, "query");
          return { message: event.message };
        },
        onDone: {
          target: "Queried",
          actions: assign({ /*...*/ }),
        },
        onError: {
          target: "Encoded",
          actions: assign({ /*...*/ }),
        },
      },
    },
    Queried: {
      always: { target: "Decoded", actions: "decodeMessage" },
      on: {
        query: "Query",
        encode: { /*...*/ },
      },
    },
    Decoded: {
      always: { target: "Reset", actions: "reset" },
      on: {
        query: "Query",
        encode: { /*...*/ },
      },
    },
    Reset: {
      always: "Idle",
    },
  },
});
```

This state machine covers every viable state and its transition in the flow. To visualize this (with the help of the [xstate visualization tool](https://stately.ai/viz)):



And in the component, you can use the state machine like the following:


```
const [state, send] = useMachine(chatMachine);
<button
  onClick={() => {
    send({ type: "encode", input: "user input" });
  }}
>
  Send
</button>
```
Overall, XState seems like a suitable choice for managing the state in the application, as it allows for modelling the flow and transitions declaratively between different states, providing a clear and maintainable solution for complex logic scenarios.