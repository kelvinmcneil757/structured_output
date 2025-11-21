# Invoice Parsing Demo with OpenAI Structured Outputs

This project demonstrates how to extract structured data from a PDF commercial invoice using OpenAI's Large Language Models (LLMs). It compares two approaches to parsing unstructured data: a naive prompt-based approach and a robust schema-based approach using OpenAI's Structured Outputs.

## Overview

The Jupyter notebook `invoice_llm_parsing_demo.ipynb` walks through the following steps:

1.  **Load PDF with OCR**: Reads text from a sample commercial invoice PDF (`Dessert.Invoice.Example.pdf`) using OCR (Optical Character Recognition) to handle image-based documents.
2.  **Naive Extraction**: Attempts to extract data into JSON format using a standard prompt. This shows how the output can be inconsistent or loosely typed without a schema.
3.  **Structured Extraction**: Defines a strict data schema using **Pydantic** models (`Invoice` and `LineItem`) and uses OpenAI's `client.beta.chat.completions.parse` method to guarantee the output matches this schema exactly.
4.  **Data Processing**: Converts the structured output into a Python dictionary and a Pandas DataFrame for easy analysis or export to CSV/Excel.

## Prerequisites

*   **Python 3.10+**
*   **OpenAI API Key**: You must have an `OPENAI_API_KEY` environment variable set.

## Dependencies

The following Python packages are required:

*   `openai`: For interacting with the OpenAI API.
*   `pydantic`: For defining data schemas.
*   `pandas`: For tabular data manipulation.
*   `pdf2image`: For converting PDF pages to images.
*   `pytesseract`: For extracting text from images (OCR).
*   `pillow`: Python Imaging Library used by pdf2image.

You can install them via pip:

```bash
pip install openai pydantic pandas pdf2image pytesseract pillow
```

## System Dependencies

Since we are using OCR, you need to install the following system tools:

*   **Poppler**: Required by `pdf2image` to process PDFs.
*   **Tesseract**: The OCR engine required by `pytesseract`.

**macOS (Homebrew):**
```bash
brew install poppler tesseract
```

**Linux (Debian/Ubuntu):**
```bash
sudo apt-get install poppler-utils tesseract-ocr
```

## Usage

1.  Ensure your OpenAI API key is set in your environment.
2.  Open `invoice_llm_parsing_demo.ipynb` in VS Code or Jupyter Lab.
3.  Run the cells in order.
4.  Observe the difference between the raw JSON string returned in the naive approach and the validated `Invoice` object returned in the structured approach.

## Key Concepts

*   **JSON Mode**: Ensures the model outputs valid JSON, but doesn't validate the *structure* of that JSON.
*   **Structured Outputs**: Uses a Pydantic model to enforce a specific schema, ensuring fields like `total_amount` are numbers and `line_items` is a list of objects with specific fields.

## Lessons Learned: OCR for Image-Based PDFs

During the development of this demo, we encountered a common real-world challenge: **not all PDFs are created equal**.

*   **The Problem**: Initially, we used standard PDF text extraction libraries (`PyPDF2`). This worked fine for "digital-native" PDFs where text is selectable. However, when we switched to a scanned invoice or a flattened PDF (like `Dessert.Invoice.Example.pdf`), the extraction returned empty strings or garbage characters. The PDF contained an *image* of text, not the text itself.

*   **The Solution**: We had to pivot to an **OCR (Optical Character Recognition)** pipeline.
    1.  **Convert to Image**: First, we use `pdf2image` to turn each page of the PDF into a high-resolution image.
    2.  **Read the Text**: Then, we pass those images to `pytesseract` (a wrapper for Google's Tesseract engine), which "reads" the pixels and outputs the raw string data.

*   **Configuration Gotchas**: Getting OCR to work isn't just `pip install`. It requires external system binaries (`poppler` and `tesseract`). On macOS, we specifically had to point the Python libraries to the Homebrew installation paths (`/opt/homebrew/bin`) to avoid `FileNotFoundError` and `TesseractNotFoundError`.

This hybrid approach—OCR for extraction + LLM for structuring—is incredibly powerful because it allows you to process virtually *any* document, whether it's a pristine digital export or a grainy scan from a fax machine.
