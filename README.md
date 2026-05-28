# Automated_Construction_of_a_Han-Nom_Literature_Corpus

## 📝 Introduction
This notebook provides a comprehensive OCR pipeline to automate the process of recognizing Sino-Nom characters and extracting text for 3 historical books: **Chinh Phu Ngam**, **Truyen Ky Man Luc**, and **Chi Nam Ngoc Am Giai Nghia**. Users only need to change the book name and page number configuration at the beginning of the file, the system will automatically call the corresponding processing functions.

## 🛠️ System Requirements & Installation
To run this notebook, you need to install the following Python libraries in `requirements.txt`:
* **Image and data processing:** `opencv-python` (`cv2`), `numpy`, `pandas`, `matplotlib`, `Pillow` (`PIL`).
* **Network communication and APIs:** `requests`, `python-dotenv`.
* **Specialized libraries (depending on the book):** 
  * `llama-parse` (used for the Truyen Ky Man Luc book).
  * `openai` (used for the Fallback mechanism calling the Qwen model via OpenRouter).

**API Keys Configuration (`.env` environment variables):**
The system uses the `.env` file to load API keys automatically:
* `LLAMA_CLOUD_API_KEY`: Required if you want to run OCR for the *Truyen Ky Man Luc* book.
* `OPENROUTER_API_KEY`: Required if you want to use the fallback mechanism to OpenRouter when the main API fails.

## 📂 Standard Directory Structure
Before running, please ensure your working directory has the following structure:
* `./images/`: Directory containing input images (`.png` or `.jpg`) and bounding box label files (`.json` if the book is **Chi Nam Ngoc Am Giai Nghia**).
* `./outputs/`: Destination directory the system will automatically create to save output results (Excel, CSV, Markdown, depending on the books).

## 🚀 Usage Instructions
In **Cell 4 (Run Configuration)** of the notebook, you adjust the 2 most important variables:

```python
BOOK = "Chỉ Nam Ngọc Âm Giải Nghĩa"  # Choose 1 of 3: "Chinh Phụ Ngâm", "Truyền Kỳ Mạn Lục", "Chỉ Nam Ngọc Âm Giải Nghĩa"
PAGE_NUMBER = "014"                  # Page name/number corresponding to the input image file (e.g., "014", "165", "page_172")
```

## ⚙️ Detailed Pipeline For Each Book

The system is designed flexibly with separate processing pipelines for each book structure within the `CONFIGS` variable:

### 1. "Chinh Phu Ngam" Book
* **Input:** Image file in `.png` format.
* **Operating method:** Uses the CLC Sino-Nom API from the KimHanNom server to upload the image and recognize characters.
* **Output:** The recognition results (bounding boxes and OCR characters) are appended to the `ChinhPhuNgam.xlsx` file located in the `./outputs/chinh_phu_ngam` directory.

### 2. "Truyen Ky Man Luc" Book
* **Input:** Image file in `.png` format.
* **Operating method:** Uses the `LlamaParse` service with the automatic table and image analysis mode (`auto_mode`).
* **Output:** The OCR data is exported in Markdown format (`.md`) and simultaneously written to the `TruyenKyManLuc.xlsx` file located in the `./outputs/truyen_ky_man_luc` directory.

### 3. "Chi Nam Ngoc Am Giai Nghia" Book
* **Input:** Image file in `.jpg` format and coordinate file in `.json` format (from LabelMe).
* **Operating method:**
  * This is a book with a complex layout. The pipeline will read the coordinates of the large character columns ("đại tự") from the JSON file, then automatically interpolate/calculate the coordinates of the small character columns ("tiểu tự") located above or below based on the `small_char_dim` parameter.
  * Crops the image according to each coordinate block (`warp_perspective_crop`).
  * Sends each cropped image block to the CLC Sino-Nom API for OCR.
  * **Fallback Mechanism (Backup):** If the main API returns empty results or the block ID is in the mandatory list, the system will automatically call the OpenRouter API (model `qwen/qwen3-vl-32b-instruct`) to translate the image.
* **Output:** Generates 2 files including `boxes.csv` (storing the coordinate list) and `pipeline_results.csv` (storing the recognized Sino-Nom character results) in the `./outputs/chi_nam_ngoc_am` directory. It supports pausing (sleep) between API calls to avoid rate limits.
