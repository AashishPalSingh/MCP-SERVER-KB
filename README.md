# OpenAI Integration with MCP

This section demonstrates how to integrate the Model Context Protocol (MCP) with OpenAI's API to create a system where OpenAI can access and use tools provided by your MCP server.

## Overview

This example shows how to:

1. Create an MCP server that exposes a knowledge base tool
2. Connect OpenAI to this MCP server
3. Allow OpenAI to dynamically use the tools when responding to user queries

## Connection Methods

This example uses the **sse transport** for communication between the client and server, which means:

- The client and server run in the different process

### Data Flow Explanation

1. **User Query**: The user sends a query to the system (e.g., "What is our company's vacation policy?")
2. **OpenAI API**: OpenAI receives the query and available tools from the MCP server
3. **Tool Selection**: OpenAI decides which tools to use based on the query
4. **MCP Client**: The client receives OpenAI's tool call request and forwards it to the MCP server
5. **MCP Server**: The server executes the requested tool (e.g., retrieving knowledge base data)
6. **Response Flow**: The tool result flows back through the MCP client to OpenAI
7. **Final Response**: OpenAI generates a final response incorporating the tool data

## How OpenAI Executes Tools

OpenAI's function calling mechanism works with MCP tools through these steps:

1. **Tool Registration**: The MCP client converts MCP tools to OpenAI's function format
2. **Tool Choice**: OpenAI decides which tools to use based on the user query
3. **Tool Execution**: The MCP client executes the selected tools and returns results
4. **Context Integration**: OpenAI incorporates the tool results into its response

## The Role of MCP

MCP serves as a standardized bridge between AI models and your backend systems:

- **Standardization**: MCP provides a consistent interface for AI models to interact with tools
- **Abstraction**: MCP abstracts away the complexity of your backend systems
- **Security**: MCP allows you to control exactly what tools and data are exposed to AI models
- **Flexibility**: You can change your backend implementation without changing the AI integration

## Implementation Details

### Server (`server.py`)

The MCP server exposes a `get_knowledge_base` tool that retrieves Q&A pairs from a JSON file.

### Client (`client.py`)

The client:

1. Connects to the MCP server
2. Converts MCP tools to OpenAI's function format
3. Handles the communication between OpenAI and the MCP server
4. Processes tool results and generates final responses

### Knowledge Base (`data/kb.json`)

Contains Q&A pairs about company policies that can be queried through the MCP server.

### Running the Example

## Server
`docker build -t ashujss11/mcp-server .`

`docker run -p 8050:8050 -d --name mcp-server ashujss11/mcp-server`

1. Ensure you have the required dependencies installed
2. Set up your OpenAI API key in the `.env` file
3. Run the client: `python client.py`

Note: With the stdio transport used in this example, you don't need to run the server separately as the client will automatically start it.



### RESPONSE

`
Initialized SSE client...
Listing tools...

Connected to server with tools: ['get_knowledge_base']

Query: What is our company's vacation policy?
-------START AI Message-------
[
  {
    'role': 'user',
    'content': "What is our company's vacation policy?"
  },
  ChatCompletionMessage(content=None,
  refusal=None,
  role='assistant',
  annotations=[
    
  ],
  audio=None,
  function_call=None,
  tool_calls=[
    ChatCompletionMessageToolCall(id='call_OrCTQBx5Xhsubq2eXge662aU',
    function=Function(arguments='{}',
    name='get_knowledge_base'),
    type='function')
  ]),
  {
    'role': 'tool',
    'tool_call_id': 'call_OrCTQBx5Xhsubq2eXge662aU',
    'content': "Here is the retrieved knowledge base:\n\nQ1: What is our company's vacation policy?\nA1: Full-time employees are entitled to 20 paid vacation days per year. Vacation days can be taken after completing 6 months of employment. Unused vacation days can be carried over to the next year up to a maximum of 5 days. Vacation requests should be submitted at least 2 weeks in advance through the HR portal.\n\nQ2: How do I request a new software license?\nA2: To request a new software license, please submit a ticket through the IT Service Desk portal. Include the software name, version, and business justification. Standard software licenses are typically approved within 2 business days. For specialized software, approval may take up to 5 business days and may require department head approval.\n\nQ3: What is our remote work policy?\nA3: Our company follows a hybrid work model. Employees can work remotely up to 3 days per week. Remote work days must be coordinated with your team and approved by your direct manager. All remote work requires a stable internet connection and a dedicated workspace. Core collaboration hours are 10:00 AM to 3:00 PM EST.\n\nQ4: How do I submit an expense report?\nA4: Expense reports should be submitted through the company's expense management system. Include all receipts, categorize expenses appropriately, and add a brief description for each entry. Reports must be submitted within 30 days of the expense. For expenses over $100, additional documentation may be required. All reports require manager approval.\n\nQ5: What is our process for reporting a security incident?\nA5: If you discover a security incident, immediately contact the Security Team at security@company.com or call the 24/7 security hotline. Do not attempt to investigate or resolve the incident yourself. Document what you observed, including timestamps and affected systems. The Security Team will guide you through the incident response process and may need your assistance for investigation.\n\n"
  }
]
-------END AI Message-------

Response: Our company's vacation policy is as follows:

- Full-time employees are entitled to 20 paid vacation days per year.
- Vacation days can be taken after completing 6 months of employment.
- Unused vacation days can be carried over to the next year, with a maximum carryover of 5 days.
- Vacation requests should be submitted at least 2 weeks in advance through the HR portal.
`