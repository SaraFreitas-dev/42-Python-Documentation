```bash
NAME = fly_in.py
PYTHON = python3
VENV = venv
PIP = $(VENV)/bin/python3 -m pip

LEVEL = easy
MAP ?= 01_linear_path.txt

all: install run

# INSTALL VENV
venv:
	@if [ ! -d "$(VENV)" ]; then \
		echo "🛠️  Creating virtual environment..."; \
		$(PYTHON) -m venv $(VENV); \
		echo "✅ Venv created successfully"; \
		echo ""; \
		echo "📌 To activate the virtual environment run:"; \
		echo "source $(VENV)/bin/activate"; \
	else \
		echo "⚠️ Venv already exists"; \
	fi

# INSTALL ALL REQUIREMENTS

install: venv
	@echo "📦 Installing requirements..."
	@$(PIP) install --upgrade pip
	@$(PIP) install -r requirements.txt
	@echo "✅ Installation complete"

# RUN THE GAME
run:
	$(VENV)/bin/python3 $(NAME) assets/maps/$(LEVEL)/$(MAP)

# DEBUGGER
debug:
	@$(VENV)/bin/python3 -m pdb $(NAME) assets/maps/$(LEVEL)/$(MAP)

# CHECK FOR NORM ERRORS
lint:
	@echo "🔍 Running flake8 and mypy..."
	$(VENV)/bin/flake8 .
	$(VENV)/bin/mypy . \
		--warn-return-any \
		--warn-unused-ignores \
		--ignore-missing-imports \
		--disallow-untyped-defs \
		--check-untyped-defs
	@echo "✅ Lint completed"

lint-strict:
	@echo "🧠 Running strict checks..."
	$(VENV)/bin/flake8 .
	$(VENV)/bin/mypy . \
		--strict
	@echo "✅ Strict lint completed"

# CLEANERS
clean:
	@echo "🧹 Cleaning cache files..."
	find . -name "__pycache__" -exec rm -rf {} +
	find . -name "*.pyc" -delete
	rm -rf .mypy_cache
	@echo "✅ Partial clean complete"

fclean: clean
	@echo "💣 Removing virtual environment..."
	rm -rf $(VENV)
	@echo "✅ Full clean complete"

re: fclean install

.PHONY: all venv install run debug lint clean fclean re
```