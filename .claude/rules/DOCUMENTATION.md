## Documentation:

### Rust Programming Language

#### 1. Write a Deterministic, Action-Oriented Summary

- **Do:** Start with a strong, action-oriented verb. Be explicit about exactly what the tool accomplishes.
- **Don't:** Use conversational filler or vague terms like "This function might help with..."
- **Example:** `/// Queries the internal HR database to retrieve a given employee's current vacation balance.`

#### 2. Document Every Parameter Thoroughly

- Use standard Markdown lists to define each parameter.
- Explicitly state the expected formats (e.g., ISO 8601 for dates, specific currency codes).
- Mention constraints directly in the text, even if the Rust type system enforces them, so the LLM doesn't waste tokens hallucinating bad inputs.

#### 3. Leverage Rust Enums for Bounded Choices

- Documenting the enum and its variants ensures the generated JSON Schema restricts the AI to specific string literals.
- Add `///` comments to each variant so the AI understands the nuance of _why_ it should choose one over the other.

#### 4. Utilize Standard rustdoc Sections for State Management

- **`# Errors`:** Explicitly document the failure states. If the AI knows _why_ a function might fail (e.g., "Returns a 404 if the user ID does not exist"), it can formulate a better fallback plan or error message for the end user.
- **`# Panics`:** Document any panics to ensure developers wrapping the skill know how to handle fatal state errors safely.

#### 5. Provide Examples (Doc Tests)

- Use standard Rust doc tests (````rust`) to demonstrate how the skill is initialized and executed. This guarantees your documentation never drifts from the actual implementation.

---

#### Example of an AI-Optimized rustdoc

````rust
/// Books a flight for the user based on the provided origin and destination.
///
/// This tool connects to the external Airline API. It should ONLY be called
/// after the user has explicitly confirmed the dates and the price.
///
/// # Arguments
///
/// * `origin_airport_code` - The 3-letter IATA code for the departure airport (e.g., "JFK", "LHR").
/// * `destination_airport_code` - The 3-letter IATA code for the arrival airport.
/// * `travel_class` - The cabin class. Must be one of the `TravelClass` enum variants.
///
/// # Errors
///
/// * Returns `FlightError::NoSeatsAvailable` if the flight is fully booked.
/// * Returns `FlightError::InvalidIataCode` if the AI provides an invalid 3-letter code.
///
/// # Examples
///
/// ```
/// # use flight_agent::{book_flight, TravelClass};
/// # tokio_test::block_on(async {
/// let result = book_flight("JFK".to_string(), "SFO".to_string(), TravelClass::Economy).await;
/// assert!(result.is_ok());
/// # });
/// ```
pub async fn book_flight(
    origin_airport_code: String,
    destination_airport_code: String,
    travel_class: TravelClass,
) -> Result<BookingConfirmation, FlightError> {
    // Implementation...
    unimplemented!()
}

/// The cabin class for the flight booking.
#[derive(Debug, Deserialize, Serialize)]
pub enum TravelClass {
    /// Standard economy seating. Use this as the default unless the user specifies otherwise.
    Economy,
    /// Business class seating. Only use if the user explicitly requests premium seating.
    Business,
}

````

### Python
**Documentation setup: Folder Structure**

```
my_python_project/
├── docs/                   # Markdown documentation directory
│   ├── guides/             # Tutorial and user guides
│   │   └── usage.md
...
├── src/
│   └── my_package/
├── mkdocs.yml              # MkDocs config file
...
```

**MKDocs configuration file Best Practice**

```yaml
site_name: My Python Project
site_description: "Awesome Python package documentation"
site_author: "Your Name"
repo_url: https://github.com/username/my_python_project

# Theme (Recommend Material for MkDocs)
theme:
  name: material
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.top
    - search.suggest
    - search.highlight
    - content.code.copy
  palette:
    - scheme: default
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/brightness-4
        name: Switch to light mode

# Navigation Structure
nav:
  - Home: index.md
  - User Guide:
      - Installation: guides/installation.md
      - Usage: guides/usage.md
  - API Reference:
      - Calculator: api/calculator.md

# Markdown extension (Must have)
markdown_extensions:
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.inlinehilite
  - pymdownx.snippets
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
  - admonition
  - pymdownx.details

# Plugins (Python API auto creation)
plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          paths: [src] # Source location
          options:
            docstring_style: google # Recommend Google style Docstring
            show_source: true
```

**Reference for documentation in the source code (`src/my_package/calculator.py`)**

````python
def add(a: int, b: int) -> int:
    """Add two integers.

    Args:
        a (int): the first number.
        b (int): the second number.

    Returns:
        int: Sum of the two numbers

    Example:
        ```python
        from my_package.calculator import add
        result = add(2, 3)
        print(result) # 5
        ```
    """
    return a + b
````

**Reference for documentation in the markdown file(`docs/api/calculator.md`)**

```markdown
# Calculator API Reference

This module provide core mathematic calculation.
::: my_package.calculator.add
```

**Advanced: Visual elements (Admonitions & Tabs)**

1. Admonitions
   Useful for visualize important information or warnings.

```markdown
!!! note "Note"
This feature only support for version 2.0 or later.

!!! warning "Warning"
Initialize the database before calling this function.
```

2. Content Tabs
   Useful for installation guideline or comparing source codes in different versions.

```markdown
=== "Windows"
`bash
    pip install my_package
    `
=== "macOS / Linux"
`bash
    pip3 install my_package
    `
```
