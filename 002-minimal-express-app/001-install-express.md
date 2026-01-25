## Install Express

### 1. Before Installing Express

Before installing Express, let’s make sure your system is ready.

---

#### Check 1: Node.js Is Installed

Open your terminal and run:

    node -v

If you see something like:

    v24.x.x

Good — Node.js is installed.

---

#### Check 2: npm Is Available

Now run:

    npm -v

If you see a version number like:

    9.x.x

Good — npm is available.

**What is npm?**

- npm = **Package Manager for Node**
- It downloads and manages libraries like **Express** for us

---

### 2. Installing Express

Now we’ll install Express step by step.

---

#### Step A: Create a Project Folder

Run the following commands:

    mkdir express-app
    cd express-app

You are now inside your **project folder**.

---

#### Step B: Create `package.json`

Run:

    npm init -y

This creates a file called:

- `package.json`

What this file means:

- “This folder is a **Node.js project**”
- It keeps track of:
  - Project info
  - Installed libraries
  - Scripts

---

#### Step C: Install Express

Now run:

    npm install express

What just happened?

- npm downloaded **Express**
- Saved it inside the `node_modules/` folder
- Added Express to `package.json` automatically

---

### 3. Verify Express Is Installed

Open the `package.json` file.

You should see something like:

    "dependencies": {
      "express": "^4.18.2"
    }

This confirms **Express is installed correctly** and ready to use.
