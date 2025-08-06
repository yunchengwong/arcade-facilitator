## Conversational Agents: Managing Environments

1. Getting started with Conversational Agents
	- https://conversational-agents.cloud.google.com/projects
	- `Create agent` > Build your own
		- Display name: Flight Booker - Env Mgt
		- Location: us-east1
		- Time zone: GMT-5:00
		- Conversation start: Flow
	- `⚙️` > General > Logging settings
		- Enable Cloud Logging
		- Enable conversation history

2. Importing a .blob virtual agent file
	- https://storage.googleapis.com/spls/DialogflowCX_agents/gsp929-start-agent.blob
	- 🔃 > `Restore agent` > Upload

3. Testing in the Draft environment
	- `🗨️` (Toggle Simulator)
		- Environment (default): Draft
		- Start Resource (default): Default Start Flow
		- Talk to agent: i want to book a flight

4. Testing in QA environment
	- Versions > Flows > Default Start Flow > `Create`
		- Version name: Flight booker main v1 chat bot
	- Environments > `Create`
		- Display Name: QA
		- Flow: Flight booker main v1 chat bot
	- `🗨️` (Toggle Simulator)
		- Environment: QA
		- Talk to agent: i want to book a flight

5. Testing in Dev environment
	- Flows > Ticket Information > `Edit fulfillment` > Agent responses > `Add dialogue response` > Agent dialogue
		- I'll be happy to assist you with that.
	- Versions > Flows > Default Start Flow > `Create`
		- Version name: Flight booker main v2 chat bot
		- Description: Version 2 adds a friendly greeting before prompting for flight details.
	- Environments > `Create`
		- Display Name: Dev
		- Flow: Flight booker main v2 chat bot
	- `🗨️` (Toggle Simulator)
		- Environment: Dev
		- Talk to agent: i want to book a flight

## Getting Started with Liquid to Customize the Looker User Experience

- Develop > Development mode (toggle)
- Develop > qwiklabs-ecommerce (project) > views > `users.view` / `order_items.view`
	- Save Changes
	- Validate LookML
	- Commit Changes & Push
	- Deploy to Production

```
view: users {
  ...

  dimension: city {
    ...
  }

  dimension: city_link {
    type: string
    sql: ${TABLE}.city ;;
    link: {
      label: "Search the web"
      url: "http://www.google.com/search?q={{ value | url_encode }}"
      icon_url: "http://www.google.com/s2/favicons?domain=www.{{ value | url_encode }}.com"
    }
  }

  dimension: order_history_button {
    label: "Order History"
    sql: ${TABLE}.id ;;
    html: <a href="/explore/training_ecommerce/order_items?fields=order_items.order_item_id, users.first_name, users.last_name, users.id, order_items.order_item_count, order_items.total_revenue&f[users.id]={{ value }}"><button>Order History</button></a> ;;
  }

  ...

  dimension: state {
    ...
  }
  
  dimension: state_link {
    type: string
    sql: ${TABLE}.state ;;
    map_layer_name: us_states
    html: {% if _explore._name == "order_items" %}
          <a href="/explore/training_ecommerce/order_items?fields=order_items.detail*&f[users.state]= {{ value }}">{{ value }}</a>
        {% else %}
          <a href="/explore/training_ecommerce/users?fields=users.detail*&f[users.state]={{ value }}">{{ value }}</a>
        {% endif %} ;;
  }
  
  ...
}
```

```
view: order_items {
  ...

  measure: total_revenue {
    ...
  }
  
  measure: total_revenue_conditional {
    type: sum
    sql: ${sale_price} ;;
    value_format_name: usd
    html: {% if value > 1300.00 %}
          <p style="color: white; background-color: ##FFC20A; margin: 0; border-radius: 5px; text-align:center">{{ rendered_value }}</p>
          {% elsif value > 1200.00 %}
          <p style="color: white; background-color: #0C7BDC; margin: 0; border-radius: 5px; text-align:center">{{ rendered_value }}</p>
          {% else %}
          <p style="color: white; background-color: #6D7170; margin: 0; border-radius: 5px; text-align:center">{{ rendered_value }}</p>
          {% endif %}
          ;;
  }
  
  ...
}
```

## A Tour of Cloud Networking

#### Task 2. Understand Virtual Private Cloud (VPC)

1. **Which feature of Google Cloud's VPC allows you to define your own private IP address range, ensuring no overlap with other networks?**

Private IP address space

2. **Which feature of Google Cloud's VPC enables you to divide your VPC into multiple subnets to organize and segment your network resources?**

Subnetwork

3. **Which feature of Google Cloud's VPC provides control over how traffic flows within your VPC and between VPCs?**

Customizable routing

4. **Which VPC configuration allows you to use RDMA for GPU communication**

Network profiles

#### Task 3. Network services

1. **Which Google Cloud network service distributes incoming traffic across multiple instances of an application or service to ensure high availability and scalability?**

Cloud Load Balancing

2. **Which Google Cloud network service translates domain names into IP addresses, enabling users to access websites and services seamlessly?**

Cloud DNS

3. **Which Google Cloud network service accelerates content delivery to users worldwide by caching content in edge locations close to their devices?**

Cloud CDN

#### Task 4. Network connectivity

1. **Which Google Cloud network connectivity solution establishes secure encrypted connections between on-premises networks and VPCs, enabling hybrid cloud deployments?**

Cloud VPN

2. **Which Google Cloud network connectivity solution provides high-bandwidth, low-latency connectivity between on-premises networks and VPCs, ideal for mission-critical applications?**

Cloud Interconnect

#### Task 5. Network security

1. **Which Google Cloud network security solution safeguards applications and websites against denial-of-service (DoS) attacks and other malicious traffic?**

Cloud Armor

2. **Which Google Cloud network security solution continuously monitors network traffic for suspicious activity, enabling early detection of potential threats?**

Cloud IDS

#### Task 6. Explore network observability

1. **Which Network Intelligence Center tool enables you to visualize the topology of your Virtual Private Cloud (VPC) networks and their associated metrics?**

Network Topology

2. **Which Network Intelligence Center tool enables you to test network connectivity to and from your VPC network?**

Connectivity test

3. **Which Network Intelligence Center tool enables you to analyze your VPC Flow Logs?**

Flow Analyzer

#### Task 7. Network Service Tiers

1. **Which Network Service Tier provides a global network with low latency, high availability and scalability, making it ideal for production workloads and demanding applications?**

Premium Tier

2. **Which Network Service Tier utilizes a regional network with lower costs, making it suitable for development, testing, and non-production environments?**

Standard Tier

## Skills Boost Arcade Trivia August 2025 Week 1

1. **Which feature in Dialogflow CX allows you to group different versions of your virtual agent's flows, pages, and intents together for different development stages or feature releases?**

Environments

2. **In Dialogflow CX, what specific component allows you to save a snapshot of your virtual agent's current state (flows, pages, intents) for deployment to different environments?**

Agent Version

3. **In Looker, what templating language is primarily used to dynamically customize the user experience, such as generating links or conditional content?**

Liquid

4. **Which specific type of Liquid tag is used to display the value of an object or variable in Looker?**

Output tag

5. **In Google Cloud, what is the global service that provides networking for Compute Engine virtual machine (VM) instances, containers, and serverless workloads?**

Virtual Private Cloud (VPC)

6. **What Google Cloud service helps distribute user traffic across multiple instances to ensure high availability and performance for applications?**

Cloud Load Balancing
