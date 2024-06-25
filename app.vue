<template>
  <div>
    <h1>vee-validate array fields</h1>

    <form @submit.prevent="handleSubmit(onSubmit)">
      <div v-for="(user, idx) in users" :key="idx" class="InputGroup">
        <div>
          <legend>User #{{ idx + 1 }}</legend>
          <label :for="`name_${idx}`">Name</label>
          <input
            :id="`name_${idx}`"
            v-model="user.name"
            @blur="validateField('name', idx)"
          />
          <span
            v-if="errors.users && errors.users[idx] && errors.users[idx].name"
            >{{ errors.users[idx].name }}</span
          >
        </div>
        <div>
          <label :for="`email_${idx}`">Email</label>
          <input
            :id="`email_${idx}`"
            v-model="user.email"
            type="email"
            @blur="validateField('email', idx)"
          />
          <span
            v-if="errors.users && errors.users[idx] && errors.users[idx].email"
            >{{ errors.users[idx].email }}</span
          >
        </div>
        <button type="button" @click="removeUser(idx)">X</button>
      </div>

      <button type="button" @click="addUser">Add User +</button>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";
import * as yup from "yup";

// Define your schema
const schema = yup.object().shape({
  users: yup
    .array()
    .of(
      yup.object().shape({
        name: yup.string().required("Name is required").label("Name"),
        email: yup
          .string()
          .email("Email is invalid")
          .required("Email is required")
          .label("Email"),
      })
    )
    .required()
    .min(1, "At least one user is required"),
});

// Initial form data
const initialData = ref({
  users: [{ name: "", email: "" }],
});

// Users state
const users = ref([...initialData.value.users]);

// Errors state
const errors = reactive({});

// Validate a specific field
const validateField = async (field, idx) => {
  try {
    await schema.validateAt(`users[${idx}].${field}`, { users: users.value });
    if (errors.users && errors.users[idx]) {
      delete errors.users[idx][field];
      if (Object.keys(errors.users[idx]).length === 0) {
        delete errors.users[idx];
      }
    }
  } catch (err) {
    if (!errors.users) {
      errors.users = {};
    }
    if (!errors.users[idx]) {
      errors.users[idx] = {};
    }
    errors.users[idx][field] = err.message;
  }
};

// Handle form submission
const handleSubmit = async (callback) => {
  try {
    await schema.validate({ users: users.value }, { abortEarly: false });
    errors.users = [];
    callback({ users: users.value });
  } catch (err) {
    errors.users = [];
    err.inner.forEach((validationError) => {
      const path = validationError.path.split(".");
      const index = parseInt(path[1], 10);
      const field = path[2];
      if (!errors.users[index]) {
        errors.users[index] = {};
      }
      errors.users[index][field] = validationError.message;
    });
  }
};

// Add a new user
const addUser = () => {
  users.value.push({ name: "", email: "" });
};

// Remove a user
const removeUser = (index) => {
  users.value.splice(index, 1);
};

// Handle form submission callback
const onSubmit = (values) => {
  alert(JSON.stringify(values, null, 2));
};
</script>

<style>
#app {
  font-family: Arial, Helvetica, sans-serif;
  max-width: 500px;
}

input {
  display: block;
}

span {
  display: block;
  margin-bottom: 20px;
  color: red;
}

label {
  display: block;
  margin-top: 20px;
}

button {
  display: block;
}

button[type="submit"] {
  margin-top: 10px;
}

form {
  padding: 20px;
  border: 1px solid black;
}

form + form {
  margin-top: 20px;
}

.InputGroup {
  padding: 10px;
  border: 2px dotted black;
  margin-bottom: 30px;
  position: relative;
}

.InputGroup button {
  position: absolute;
  top: 10px;
  right: 10px;
}
</style>
