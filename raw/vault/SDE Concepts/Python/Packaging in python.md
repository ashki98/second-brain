# Packaging in python

# **🐍 Python: Poetry vs Virtualenv/Venv**

### **✅ Poetry Features:**

- Manages **virtual environments**
- Handles **dependency installation & locking**
- Supports **package building and publishing**
- Uses pyproject.toml for configuration

| **Feature** | virtualenv **/** venv | **Poetry** |
| --- | --- | --- |
| Virtual Environment | ✅ | ✅ |
| Dependency Management | ❌ (use pip) | ✅ |
| Dependency Locking | ❌ | ✅ (poetry.lock) |
| Project Packaging | ❌ | ✅ (via pyproject.toml) |
| Package Publishing | ❌ | ✅ (poetry publish) |
| Config Format | N/A | pyproject.toml |

# **🔁 Poetry vs Pipenv**

| **Feature** | **Pipenv** | **Poetry** |
| --- | --- | --- |
| Virtualenv Management | ✅ | ✅ |
| Dependency Management | ✅ | ✅ |
| Lock File | Pipfile.lock | poetry.lock |
| Packaging & Publishing | ❌ | ✅ |
| Uses pyproject.toml | Partial | Full Support |
| Dependency Resolver | Flaky | Reliable |
| Community Sentiment (2025) | Mixed | Positive |
| Dev Activity | Slower | Actively Maintained |

### **🧠 Philosophy:**

- **Pipenv**: Aims to simplify dev workflows (apps only)
- **Poetry**: Full packaging tool for both apps and libraries

📦 What are wheel and sdist?

🔧 Wheel (.whl):
•	A pre-built package (fast install, no build step)
•	Preferred by pip

📦 sdist (.tar.gz):
•	Source archive that must be built during install

# **🔒 Significance of Lock Files**

### **📝 What is a lock file?**

- Captures **exact versions** of all dependencies & sub-dependencies.
- Ensures everyone (devs, CI, production) uses the **same setup**.

### **📁 Examples:**

- poetry.lock (Poetry)
- Pipfile.lock (Pipenv)
- package-lock.json (npm)
- yarn.lock (Yarn)

### **✅ Benefits:**

- Reproducible installs
- Avoids surprises from automatic upgrades
- Better security auditing
- Faster installs (no resolution step)

### **🧠 Remember:**

> pyproject.toml (or Pipfile) = What you want
> 

> poetry.lock (or Pipfile.lock) = What you got
>
