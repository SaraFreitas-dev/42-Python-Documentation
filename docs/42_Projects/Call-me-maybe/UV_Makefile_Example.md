# 📦 UV & Python Makefile Example


```text
UV = uv

HF_HOME = /sgoinfre/sarfreit/hf_cache
FUNCTIONS = data/input/functions_definition.json
INPUT = data/input/function_calling_tests.json
OUTPUT = data/output/function_calling_results.json

export HF_HOME

all: install run

# INSTALL ALL REQUIREMENTS
install:
	@if command -v uv >/dev/null 2>&1; then \
		echo "✅ uv is already installed ($$(uv --version))"; \
	elif [ -f "$$HOME/.local/bin/uv" ]; then \
		echo "✅ uv found in $$HOME/.local/bin"; \
	else \
		echo "📦 uv not found. Installing..."; \
		curl -LsSf https://astral.sh/uv/install.sh | sh; \
	fi
	@echo "📦 Syncing dependencies..."
	@PATH="$$HOME/.local/bin:$$PATH" uv sync
	@echo "✅ Dependencies ready"

# RUN THE PROGRAM
run:
	@$(UV) run python -m src \
		--functions_definition $(FUNCTIONS) \
		--input $(INPUT) \
		--output $(OUTPUT)

run-verbose:
	@$(UV) run python -m src \
		--functions_definition $(FUNCTIONS) \
		--input $(INPUT) \
		--output $(OUTPUT) \
		--verbose

# DEBUG AND HELP
debug:
	@$(UV) run python -m pdb -m src \
		--functions_definition $(FUNCTIONS) \
		--input $(INPUT) \
		--output $(OUTPUT)

help:
	@echo ""
	@echo "╔══════════════════════════════════════════════════════╗"
	@echo "║              Call Me Maybe — Make Commands           ║"
	@echo "╚══════════════════════════════════════════════════════╝"
	@echo ""
	@echo "  📦 make install       Install project dependencies"
	@echo "  ▶️  make run           Run with default settings"
	@echo "  🔍 make run-verbose   Run with detailed generation trace"
	@echo "  🐞 make debug         Run with Python's pdb debugger"
	@echo ""
	@echo "  ✅ make lint          Run flake8 and mypy checks"
	@echo "  🧠 make lint-strict   Run stricter mypy checks"
	@echo ""
	@echo "  🧹 make clean         Remove cache files"
	@echo "  💣 make fclean        Remove cache + Output files + virtual environment"
	@echo "  🔁 make re            Full clean and reinstall"
	@echo ""
	@echo "  ℹ️  Program-specific options running with run python -m src ... (--model, --verbose, etc.):"
	@echo ""
	@$(UV) run python -m src --help

# CHECK FOR NORM ERRORS
lint:
	@echo "🔍 Running flake8 and mypy..."
	@$(UV) run flake8 .
	@$(UV) run mypy . \
		--warn-return-any \
		--warn-unused-ignores \
		--ignore-missing-imports \
		--disallow-untyped-defs \
		--check-untyped-defs
	@echo "✅ Lint completed"

lint-strict:
	@echo "🧠 Running strict checks..."
	@$(UV) run flake8 .
	@$(UV) run mypy . --strict
	@echo "✅ Strict lint completed"

# CLEANERS
# CLEANERS
clean:
	@echo "\n🧹 Cleaning cache files..."
	@find . -type d -name "__pycache__" -exec rm -rf {} +
	@find . -type f -name "*.pyc" -delete
	@echo "\n🧹 Cleaning generated output..."
	@rm -f data/output/*.json
	@echo "\n✅ Partial clean complete\n"

fclean: clean
	@echo "\n💣 Removing virtual environment..."
	@rm -rf .venv
	@echo "\n✅ Full clean complete\n"

re: fclean install

.PHONY: all install run debug lint lint-strict clean fclean re

```