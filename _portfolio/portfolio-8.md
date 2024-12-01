---
title: "Tea Ordering System for Bubble Tea Shop"
excerpt: "This project implements an automated tea ordering system for a bubble tea shop using modern web technologies and frameworks. The system allows customers to customize their tea order, choose toppings, and track the order status in real-time.<br/><img src='/images/system-architecture.png'>"
collection: portfolio
---
<a id="top"></a>

***Guowang Zeng, Yizhou Zhang***

## Introduction

Bubble tea, a popular drink originating from Taiwan, has gained global popularity over the years. With its customizable options, including different tea flavors, sweetness levels, and toppings, it has become a favorite among many. To streamline the ordering process and improve customer experience, we developed a Tea Ordering System for a bubble tea shop. This system leverages modern web technologies and frameworks to provide an intuitive and easy-to-use interface for customers, while also offering a backend to handle order management and status tracking.

The system allows customers to:
- Select tea type (e.g., green tea, black tea, oolong).
- Customize the sweetness and ice level.
- Choose from a variety of toppings (e.g., tapioca pearls, aloe vera, mango chunks).
- Track the status of their order (e.g., preparing, ready for pickup).

## Methodology

The system was designed with two main components: the front-end for customer interaction and the back-end for managing orders. The front-end is built using **Vue** for a responsive user interface, while the back-end utilizes **Node.js** and **Express** for handling API requests. The system uses a **MongoDB** database to store customer orders and statuses.

### Key Features:
- **Tea Customization**: Customers can choose the type of tea, sweetness level, ice level, and toppings.
- **Real-Time Order Tracking**: Customers can track the progress of their order (e.g., preparing, ready for pickup) through the user interface.
- **Backend API**: The back-end provides RESTful APIs for creating, updating, and retrieving order data.
- **Data Persistence**: All orders are stored in a MongoDB database to maintain persistent records of customer orders.

## System Architecture

The Tea Ordering System consists of the following components:
1. **Frontend (Vue)**: Provides the user interface for customers to interact with the system.
2. **Backend (Node.js/Express)**: Handles the business logic and manages customer orders.
3. **Database (MongoDB)**: Stores order data and customer information.

![System Architecture](/images/system-architecture.png)

## Order Process Flow

The order process flow is simple and intuitive:
1. **Select Tea**: The customer selects a tea base (e.g., green tea, black tea).
2. **Customize Order**: The customer can adjust sweetness, ice level, and choose toppings.
3. **Submit Order**: After customization, the customer submits the order.
4. **Track Order**: The customer can track the status of their order in real-time (e.g., preparing, ready for pickup).

## Key Technologies
- Vue: Used for building a dynamic and responsive user interface.
- Node.js: Provides a lightweight, scalable backend to handle API requests.
- Express: A minimal web framework for handling HTTP requests in Node.js.
- MongoDB: A NoSQL database used to store orders and customer data.
- Socket.IO: Used for real-time order status updates.

## Results
# User Interface
The customer interface provides a clean and simple design for making tea orders. Customers can easily navigate through the available options, select their preferences, and submit the order.


# Real-Time Order Tracking
Once an order is placed, the status is updated in real-time on the customer interface. The status changes from "Preparing" to "Ready for Pickup" once the order is complete.

## Setup Instructions
To run the Tea Ordering System locally, follow the steps below to set up the project environment:

1. MongoDB (Database)
Run MongoDB in a Docker container:

```bash
docker run -p 127.0.0.1:27017:27017 -d --rm --name mongo mongo:7.0.5
```

Code Implementation
Database (setupMongo.ts)

```typescript
import { MongoClient } from 'mongodb'
import {Operator, Customer, Ingredient} from './data'

// Connection URL
const url = 'mongodb://127.0.0.1:27017'
const client = new MongoClient(url)

const operators: Operator[] = [
  {
    _id: "jim",
    name: "Jim",
    password: "123456", 
  },
  {
    _id: "mary",
    name: "Mary",
    password: "123456",
  },
]

const customers: Customer[] = [
  {
    _id: "alice",
    name: "Alice",
    password: "123456",
  },
  {
    _id: "bob",
    name: "Bob",
    password: "123456",
  },
]

const ingredients: Ingredient[] = [
  { _id: "i1", name: "Banana", cost: 1 },
{ _id: "i2", name: "Strawberry", cost: 3 },
{ _id: "i3", name: "Blueberry", cost: 4 },
{ _id: "i4", name: "Mango", cost: 2 },
{ _id: "i5", name: "Pineapple", cost: 3 },
{ _id: "i6", name: "Spinach", cost: 2 },
{ _id: "i7", name: "Kale", cost: 3 },
{ _id: "i8", name: "Almond Milk", cost: 4 },
{ _id: "i9", name: "Greek Yogurt", cost: 3 },
{ _id: "i10", name: "Honey", cost: 2 }
  ];

async function main() {
  await client.connect()
  console.log('Connected successfully to MongoDB')

  const db = client.db("test")

  // set up unique index for upsert -- to make sure a customer cannot have more than one draft order
  db.collection("orders").createIndex(
    { customerId: 1 }, 
    { unique: true, partialFilterExpression: { state: "draft" } }
  )

  // add data
  console.log("inserting customers", await db.collection("customers").insertMany(customers as any))
  console.log("inserting operators", await db.collection("operators").insertMany(operators as any))
  console.log("inserting ingredients", await db.collection("possibleIngredients").insertMany(ingredients as any))

  process.exit(0)
}

main()
```

2. Server (Backend)
Navigate to the backend directory and install dependencies, set up the environment, and start the server:

```bash
npm install      # Install required dependencies
npm run setup    # Set up the initial database or configurations
npm start        # Start the server
```

Code Implementation
Backend (server.ts)
```typescript
import express from 'express'
import bodyParser from 'body-parser'
import pino from 'pino'
import expressPinoLogger from 'express-pino-logger'
import { Collection, Db, MongoClient, ObjectId } from 'mongodb'
import { Customer, CustomerWithOrders, DraftOrder, Operator, OperatorWithOrders, Order, Ingredient } from './data'

// set up Mongo
//for docker run
// const url = process.env.MONGO_URL || 'mongodb://127.0.0.1:27017'
//for k8s run
//const url = process.env.MONGO_URL || 'mongodb://db'
// const url = 'mongodb://127.0.0.1:27017' 
const url = process.env.MONGO_URL || 'mongodb://127.0.0.1:27017'
const client = new MongoClient(url)
let db: Db
let customers: Collection<Customer>
let orders: Collection<Order>
let operators: Collection<Operator>
let possibleIngredients: Collection<Ingredient>

// set up Express
const app = express()
const port = parseInt(process.env.PORT) || 8191
app.use(bodyParser.json())

// set up Pino logging
const logger = pino({
  transport: {
    target: 'pino-pretty'
  }
})
app.use(expressPinoLogger({ logger }))

// app routes
app.get("/api/possible-ingredientIds", async (req, res) => {
  res.status(200).json(await possibleIngredients.find({}).toArray())
})

app.get("/api/orders", async (req, res) => {
  res.status(200).json(await orders.find({ state: { $ne: "draft" }}).toArray())
})

app.get("/api/customer/:customerId", async (req, res) => {
  const _id = req.params.customerId
  const customer: Partial<CustomerWithOrders> | null = await customers.findOne({ _id })
  if (customer == null) {
    res.status(404).json({ _id })
    return
  }
  customer.orders = await orders.find({ customerId: _id, state: { $ne: "draft" } }).toArray()
  res.status(200).json(customer)
})

app.get("/api/operator/:operatorId", async (req, res) => {
  const _id = req.params.operatorId
  const operator: Partial<OperatorWithOrders> | null = await operators.findOne({ _id })
  if (operator == null) {
    res.status(404).json({ _id })
    return
  }
  operator.orders = await orders.find({ operatorId: _id }).toArray()
  res.status(200).json(operator)
})

app.get("/api/customer/:customerId/draft-order", async (req, res) => {
  const { customerId } = req.params

  // TODO: validate customerId

  const draftOrder = await orders.findOne({ state: "draft", customerId })
  res.status(200).json(draftOrder || { customerId, ingredients: [] })
})

app.put("/api/customer/:customerId/draft-order", async (req, res) => {
  const order: DraftOrder = req.body

  // TODO: validate customerId

  const result = await orders.updateOne(
    {
      customerId: req.params.customerId,
      state: "draft",
    },
    {
      $set: {
        ingredientIds: order.ingredientIds,
        sweetness: order.sweetness,
        ice: order.ice
      }
    },
    {
      upsert: true
    }
  )
  console.log(result);
  res.status(200).json({ status: "ok" })
})

app.post("/api/customer/:customerId/submit-draft-order", async (req, res) => {
  const result = await orders.updateOne(
    {
      customerId: req.params.customerId,
      state: "draft",
    },
    {
      $set: {
        state: "queued",
      }
    }
  )
  if (result.modifiedCount === 0) {
    res.status(400).json({ error: "no draft order" })
    return
  }
  res.status(200).json({ status: "ok" })
})

app.put("/api/order/:orderId", async (req, res) => {
  const order: Order = req.body

  // TODO: validate order object

  const condition: any = {
    _id: new ObjectId(req.params.orderId),
    state: { 
      $in: [
        // because PUT is idempotent, ok to call PUT twice in a row with the existing state
        order.state
      ]
    },
  }
  switch (order.state) {
    case "blending":
      condition.state.$in.push("queued")
      // can only go to blending state if no operator assigned (or is the current user, due to idempotency)
      condition.$or = [{ operatorId: { $exists: false }}, { operatorId: order.operatorId }]
      break
    case "done":
      condition.state.$in.push("blending")
      condition.operatorId = order.operatorId
      break
    default:
      // invalid state
      res.status(400).json({ error: "invalid state" })
      return
  }
  
  const result = await orders.updateOne(
    condition,
    {
      $set: {
        state: order.state,
        operatorId: order.operatorId,
      }
    }
  )

  if (result.matchedCount === 0) {
    res.status(400).json({ error: "orderId does not exist or state change not allowed" })
    return
  }
  res.status(200).json({ status: "ok" })
})

//log in api, server side handle
app.post("/api/login", async (req, res) => {
  
  const { name, password,identity } = req.body
  
  if (identity === "operator") {
    const user = await operators.findOne({ _id:name, password })
    if (user == null) {
  res.status(200).json({ status:"error", message: "invalid username or password" })
      return
    }
    res.cookie("username", user.name);
    res.cookie("userType", "operator");
    res.status(200).json({...user,userType:"operator",  status:"success",message: " Login successful! Welcome back!"})
    return
  }
  if (identity === "customer") {
    const user = await customers.findOne({ _id:name,password })
    if (user == null) {
      res.status(200).json({ status:"error", message: "invalid username or password" })
      return
    }
  
    res.cookie("username", user.name);
    res.cookie("userType", "customer");
    res.status(200).json({...user,userType:"customer",status:"success" ,message: " Login successful! Welcome back!"})
    return
  }
 
})

// connect to Mongo
client.connect().then(() => {
  console.log('Connected successfully to MongoDB')
  db = client.db("test")
  operators = db.collection('operators')
  orders = db.collection('orders')
  customers = db.collection('customers')
  possibleIngredients = db.collection('possibleIngredients')

  // start server
  app.listen(port, () => {
    console.log(`Smoothie server listening on port ${port}`)
  })
})
```

3. UI (Frontend)
Navigate to the frontend directory, install dependencies, and start the development server:
```bash
npm install      # Install required dependencies
npm run dev      # Start the frontend server in development mode
```
Code Implementation
Frontend (LoginScreen.vue)

![bubbletea_login](/images/bubbletea_login.png )

```vue
<template>
  <div class="main-container">
    <div class="login-container">
      <b-jumbotron class="custom-jumbotron">
        <template #header>
          <h1 class="text-center">Bubble Tea Bliss</h1>
        </template>
      </b-jumbotron>
      
      <b-alert :variant="variant" :show="showMessage">{{ message }}</b-alert>

      <b-card class="custom-card">
        <b-form @submit.prevent="handleLogin">
          <b-form-group label="Name" label-for="name-input">
            <b-form-input id="name-input" v-model="name" type="text" required autofocus></b-form-input>
          </b-form-group>
          <b-form-group label="Password" label-for="password-input">
            <b-form-input id="password-input" v-model="password" type="password" required></b-form-input>
          </b-form-group>
          <b-form-group label="Identity">
            <b-form-radio-group v-model="identity" name="identity">
              <b-form-radio value="operator">Operator</b-form-radio>
              <b-form-radio value="customer">Customer</b-form-radio>
            </b-form-radio-group>
          </b-form-group>
          <b-button type="submit" variant="primary" block class="custom-button login-button">Login</b-button>
        </b-form>
      </b-card>
    </div>
  </div>
</template>


<script lang="ts" setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
const router = useRouter();
const name = ref('');
const password = ref('');
const identity = ref('operator');
const showMessage = ref(false);
const message = ref('');
const variant = ref('danger');

const handleLogin = async () => {
  const res = await (await fetch("/api/login", {
    method: 'POST',
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ name: name.value, password: password.value, identity: identity.value }),
  })).json()

  if (res.status === "success") {
    showMessage.value = true;
    message.value = res.message;
    variant.value = "success";
    setTimeout(() => {
      showMessage.value = false;
      router.push(`/${res.userType}/${name.value}`);
    }, 1000);
  } else {
    showMessage.value = true;
    message.value = res.message;
    setTimeout(() => {
      showMessage.value = false;
    }, 3000);
  }
};
</script>

<style scoped>
.main-container {
  background-color: rgba(255, 255, 255, 0.85); /* White with 85% opacity */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin: 50px auto; /* Adjust top margin and center horizontally */
  max-width: 600px; /* Container width, adjust as needed */
}

.login-container {
  padding: 20px;
}

.custom-jumbotron {
  background: linear-gradient(135deg, #009efd, #FF69B4);
  color: white;
  border-radius: 10px;
  margin-bottom: 20px;
}

.custom-card {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}

.custom-button {
  background-color: #009efd;
  color: white;
  transition: transform 0.2s ease-in-out, background-color 0.2s ease;
}

.custom-button:hover {
  transform: scale(1.05);
  background-color: #FF69B4;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.login-button {
  background-color: #4CAF50; /* Green for login to highlight the primary action */
}
</style>
```

Frontend (CustomerScreen.vue)

![bubbletea_customer](/images/bubbletea_customer.png )

```vue
<template>
  <div class="main-container">
    <div class="customer">
      <div class="content">
        <b-jumbotron class="custom-jumbotron">
          <template #header>
            <h1>Welcome, {{ name }}</h1>
          </template>
        </b-jumbotron>

        <div class="button-group">
          <b-button class="custom-button refresh-button" @click="refresh">Refresh</b-button>
          <RouterLink :to="{ path: `/history/${customerId}` }">
            <b-button class="custom-button history-button">History Order</b-button>
          </RouterLink>
        </div>

        <h2 class="section-title">Preset</h2>
        <div class="button-group">
          <b-button
            class="custom-button preset-button"
            v-for="drink in drinkNames"
            :key="drink"
            @click="selectDrink(drink)"
          >
            {{ drink }}
          </b-button>
        </div>

        <h2 class="section-title">Draft Order</h2>
        <p class="bold-text">Check the ingredients you want:</p>
        <div class="card-group">
          <div
            v-for="ig in possibleIngredients"
            :key="ig['_id']"
            class="ingredient-card"
          >
            <div class="d-flex justify-content-between align-items-center">
              <span class="ingredient-name"><b>{{ ig.name }}</b></span>
              <b-button
                class="custom-button ingredient-button"
                @click="onAddIngredients(ig['name'])"
              >
                Add
              </b-button>
            </div>
          </div>
        </div>

        <div>
          <b-form-group>
            <template #label>
              <strong class="text-danger">Ice Level</strong>
            </template>
            <b-form-spinbutton v-model="ice" :min="0" :max="100" :step="25" class="custom-button"></b-form-spinbutton>
          </b-form-group>
          <b-form-group>
            <template #label>
              <strong class="text-danger">Sweetness Level</strong>
            </template>
            <b-form-spinbutton v-model="sweetness" :min="0" :max="100" :step="25" class="custom-button"></b-form-spinbutton>
          </b-form-group>
        </div>

        <h2 class="section-title">List of Added Ingredients:</h2>
        <div class="button-group">
          <b-badge
            v-for="ig in selectedIngredients"
            :key="ig['_id']"
            variant="primary"
            class="ingredient-button"
          >
            {{ ig.name }}
            <b-button
              class="custom-button remove-button"
              @click="onRemoveIngredients(ig['_id'])"
            >
              &times;
            </b-button>
          </b-badge>
        </div>

        <div class="button-group">
          <b-button class="custom-button save-button" @click="save">Save</b-button>
          <b-button
            class="custom-button submit-button"
            @click="showSubmit"
          >
            Submit
          </b-button>
        </div>
        <div class="note bold-red-note">Note: must save before submitting</div>
      </div>
      <!-- <div class="sidebar">
        <LLMChat
          :possibleIngredients="possibleIngredients"
          @wordClick="onAddIngredients"
        ></LLMChat>
      </div> -->
    </div>
  </div>

  <b-modal v-model="showDialog" title="Submit" class="custom-jumbotron">
    <div>
      <b-table
        responsive
        :items="selectedIngredients"
        :fields="sumbitTableFields"
        class="custom-button"
      ></b-table>
    </div>

    <p>Ice: {{ ice }}%</p>
    <p>Sweetness: {{ sweetness }}%</p>
    <p class="total-cost">Total Price: {{ totalPrice }}</p>

    <template #modal-footer>
      <b-button
        variant="secondary"
        @click="showDialog = false"
        class="custom-button close-button"
      >
        Close
      </b-button>
      <b-button
        variant="primary"
        @click="submit"
        class="custom-button submit-button"
      >
        Submit
      </b-button>
    </template>
  </b-modal>
</template>


<script setup lang="ts">
import {onMounted, ref, computed, Ref} from 'vue'
import { CustomerWithOrders, Ingredient, DrinkIngredients } from "../data"
import { BvTableFieldArray } from "bootstrap-vue";
// import LLMChat from './LLMChat.vue';


// props
interface Props {
  customerId: string
}

// default values for props
const props = withDefaults(defineProps<Props>(), {
  customerId: "",
})

const customer: Ref<CustomerWithOrders | null> = ref(null)

const name = computed(() => customer.value?.name || props.customerId)
const possibleIngredients: Ref<Ingredient[]> = ref([])
const selectedIngredients: Ref<Ingredient[]> = ref([])
// The name of the default ingredient
const drinkNames: Ref<string[]> = ref(["Milk Tea", "Juice"]);

// Preset drink ingredients
const drinkIngredients: Ref<DrinkIngredients> = ref({});
// Dialog display
const showDialog = ref(false)
// Sweetness
const sweetness = ref(50)
// Ice level
const ice = ref(50)
const sumbitTableFields: Ref<BvTableFieldArray> = ref([
  {
    key: '_id'
  },
  {
    key: 'name'
  },
  {
    key: 'cost'
  },
 
])
  // generate by chatgpt
  const totalPrice: Ref<number> = computed(() => {
  return selectedIngredients.value.reduce((total, ingredient) => {
  
    return total + (ingredient.cost || 0);
  }, 0);
});



async function refresh() {
  selectedIngredients.value = []  
  const result = await (await fetch("/api/possible-ingredientIds")).json()
  console.log(result);
  
  possibleIngredients.value = result
  console.log(getRandomCombinations(result, 2));
  drinkNames.value.forEach((name) => {
    const data= getRandomCombinations(result, 2);
    console.log(name,data);
    
    drinkIngredients.value[name] = result.filter((ig: { _id: string; }) => data.includes(ig._id) );
  });

  console.log(drinkIngredients.value);
  
 
}
onMounted(refresh)

async function save() {
  await fetch(
    "/api/customer/" + encodeURIComponent(props.customerId) + "/draft-order",
    {
      headers: {
        "Content-Type": "application/json",
      },
      method: "PUT",
      body: JSON.stringify({ ingredientIds: selectedIngredients.value.map(ig => ig._id), sweetness: sweetness.value, ice: ice.value})
    }
  )
}

async function submit() {
 
  await fetch(
    "/api/customer/" + encodeURIComponent(props.customerId) + "/submit-draft-order",
    { method: "POST" }
  )
  await refresh()
  showDialog.value = false
}
const showSubmit=()=>{
  showDialog.value = true
}
// generate by chatgpt
const onAddIngredients = (name: string) => {
  console.log(possibleIngredients.value);
  
  const ingredient = possibleIngredients.value.find(ig => ig.name === name);
  if (ingredient === undefined) {
    return;
  }
  selectedIngredients.value.push({ ...ingredient });
}

const onRemoveIngredients = (_id:String)=>{
  const index = selectedIngredients.value.findIndex(ig => ig._id === _id)
  if(index === -1){
    return
  }
  selectedIngredients.value.splice(index, 1)
}
// Randomly select count combinations from the original data, each combination contains a name and corresponding ID
const getRandomCombinations = (data: Ingredient[], count:number) =>{
  const selected = [];
  while (selected.length < count) {
    const randomIndex = Math.floor(Math.random() * data.length);
    selected.push( data[randomIndex]._id);
  }
  return selected;
}
const selectDrink=(drink:string)=>{


  selectedIngredients.value = [...selectedIngredients.value,  ...drinkIngredients.value[drink]  ]

}
</script>

<style scoped>
.main-container {
  background-color: rgba(255, 255, 255, 0.85); /* White with 85% opacity */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin: auto;
  max-width: 1200px; /* Adjust based on layout needs */
}

.customer {
  display: flex;
  flex-direction: row;
}

.content,
.sidebar {
  flex: 1;
  padding: 20px;
}

.custom-jumbotron {
  background: linear-gradient(135deg, #009efd, #FF69B4);
  color: white;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.section-title {
  color: #007bff;
}

.button-group,
.card-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 10px;
}

.ingredient-name {
  margin-right: 20px;
}

.ingredient-button,
.custom-button {
  transition: transform 0.2s ease-in-out, background-color 0.2s ease;
  transform: scale(1.0);
  background-color: #009efd;
  color: white;
}

.ingredient-button:hover,
.custom-button:hover {
  transform: scale(1.05);
  background-color: #FF69B4;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.refresh-button,
.save-button {
  background-color: #4CAF50;
}

.history-button,
.submit-button {
  background-color: #FFC107;
}

.ingredient-card {
  transition: transform 0.2s ease-in-out;
  width: calc(40% - 30px);
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.ingredient-card:hover {
  transform: scale(1.05);
}

.remove-button {
  background-color: #dc3545 !important;
}

.close-button {
  background-color: #6c757d !important;
}

.bold-text {
  font-weight: bold;
}

.bold-red-note {
  font-weight: bold;
  color: red; /* Or use a specific shade of red if you have a color palette */
}

@media (max-width: 768px) {
  .ingredient-card {
    width: calc(50% - 10px);
  }

  .customer {
    flex-direction: column;
  }

  .content, .sidebar {
    padding: 10px;
  }
}

@media (max-width: 480px) {
  .ingredient-card {
    width: 100%;
  }
}
</style>
```
Frontend (OperatorScreen.vue)

![bubbletea_operator](/images/bubbletea_operator.png )

```vue
<template>
  <div class="work-screen-container">
    <div class="work-screen">
      <b-jumbotron class="custom-jumbotron">
        <template #header>
          <h1>Work Screen for <strong>{{ name }}</strong></h1>
        </template>
      </b-jumbotron>
      <h2 class="section-title">Orders</h2>
      <b-button @click="refresh" class="custom-button refresh-button">Refresh</b-button>
      <div class="table-responsive">
        <b-table responsive :items="orders" :fields="fields" class="custom-table">
          <template #cell(operatorId)="cellScope">
            <span v-if="cellScope.value">
              {{ cellScope.value }}
              <b-button @click="updateOrder(cellScope.item._id, 'done')" v-if="cellScope.value === operatorId && cellScope.item.state !== 'done'" class="custom-button action-button">
                Done
              </b-button>
            </span>
            <b-button v-else @click="updateOrder(cellScope.item._id, 'blending')" class="custom-button action-button">
              Start Blending
            </b-button>
          </template>
        </b-table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, computed } from 'vue';
import { Ingredient, Operator, Order } from "../data";

interface Props {
  operatorId: string
}

const props = withDefaults(defineProps<Props>(), {
  operatorId: "",
})

const operator = ref<Operator | null>(null);
const orders = ref<Order[]>([]);
const name = computed(() => operator.value?.name || props.operatorId);
const possibleIngredients = ref<Ingredient[]>([]);

async function refresh() {
  possibleIngredients.value = await (await fetch("/api/possible-ingredientIds")).json();
  if (props.operatorId) {
    operator.value = await (await fetch("/api/operator/" + encodeURIComponent(props.operatorId))).json();
  }
  orders.value = await (await fetch("/api/orders/")).json();
}

onMounted(refresh);

const fields = ["_id", "customerId", "state", "ice", "sweetness", {
  key: 'ingredientIds',
  label: "ingredients",
  formatter: (value: string[]) => value.map(id => {
    let r = possibleIngredients.value.find(v => v._id == id);
    if (r === undefined) {
      return '';
    } else {
      return r.name;
    }
  }).filter(value => value != "").join(", ")
}, "operatorId"]

async function updateOrder(orderId: string, state: string) {
  await fetch(
    "/api/order/" + encodeURIComponent(orderId), {
      headers: {
        "Content-Type": "application/json",
      },
      method: "PUT",
      body: JSON.stringify({ operatorId: props.operatorId, state })
    }
  );
  await refresh();
}
</script>

<style scoped>
.work-screen-container {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.work-screen {
  margin: 10px;
}

.custom-jumbotron {
  background: linear-gradient(135deg, #009efd, #FF69B4);
  color: white;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.section-title {
  color: #007bff;
}

.custom-button,
.action-button {
  transition: transform 0.2s ease-in-out, background-color 0.2s ease;
  background-color: #009efd;
  color: white;
}

.custom-button:hover,
.action-button:hover {
  transform: scale(1.05);
  background-color: #FF69B4;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.refresh-button {
  background-color: #4CAF50;
}

.table-responsive {
  margin-top: 20px;
}

.custom-table {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.action-button {
  background-color: #007bff !important;
}

.action-button:hover {
  background-color: #0056b3 !important;
}
</style>
```
Frontend (App.vue)
```vue
<template>
  <router-view />
</template>
<style>
.global-word {
  background-color: #007bff; /* Replace with the actual color value */
  cursor: pointer;
}
</style>
```

4. Tests (Optional)
If you want to run tests using Playwright, navigate to the tests directory and run the following command:
```bash
npx playwright test
```
Code Implementation
Test (CustomerTest1.spec.js)
```javascript
import { test, expect, chromium } from '@playwright/test';

test.use({ timeout: 100000 });

test('test', async ({ page }) => {
    const browser = await chromium.launch({ headless: false, slowMo: 450 });
    const context = await browser.newContext();
    page = await context.newPage();


    await page.goto('http://localhost:8190/login');
    await page.getByLabel('Name').click();
    await page.getByLabel('Name').press('Meta+c');
    await page.getByLabel('Name').fill('bob');
    await page.getByLabel('Name').press('Tab');
    await page.getByLabel('Password').fill('123456');
    await page.getByText('Customer').click();
    await page.getByLabel('Customer').press('Meta+c');
    await page.getByRole('button', { name: 'Login' }).click();
    await page.getByRole('button', { name: 'Juice' }).click();
    await page.locator('.d-flex > .btn').first().click();
    await page.locator('.d-flex > .btn').first().press('Meta+c');
    await page.locator('div:nth-child(2) > .d-flex > .btn').first().click();
    await page.getByRole('button', { name: 'Refresh' }).click();
    await page.getByRole('button', { name: 'Juice' }).click();
    await page.getByRole('group', { name: 'Ice Level' }).getByLabel('Decrement').click();
    await page.getByRole('group', { name: 'Ice Level' }).getByLabel('Decrement').press('Meta+c');
    await page.getByRole('group', { name: 'Sweetness Level' }).getByLabel('Decrement').click();
    await page.getByRole('group', { name: 'Sweetness Level' }).getByLabel('Decrement').press('Meta+c');
    await page.getByRole('group', { name: 'Sweetness Level' }).getByLabel('Decrement').click();
    await page.getByRole('button', { name: 'Save' }).click();
    await page.getByRole('button', { name: 'Submit' }).click();
    await page.getByText('Close').click();
    await page.getByRole('button', { name: 'Submit' }).click();
    await page.getByLabel('Submit').getByRole('button', { name: 'Submit' }).click();
    await page.getByRole('button', { name: 'History Order' }).click();
    await page.goto('http://localhost:8190');
    
});
```
## Conclusion

This Tea Ordering System enhances the customer experience by providing an easy-to-use interface for tea customization and order management. The real-time order tracking feature ensures that customers are always informed about the status of their order. The backend ensures smooth order processing and data persistence, while the frontend offers a clean and responsive interface.

The system can be expanded to include additional features, such as payment integration, loyalty programs, and inventory management, making it a flexible solution for future enhancements.








