import os
import shutil
import hashlib
import logging
from pathlib import Path

# =========================
# CONFIG
# =========================

FILE_TYPES = {
    "Images": [".jpg", ".jpeg", ".png", ".gif", ".webp"],
    "Videos": [".mp4", ".mkv", ".avi", ".mov"],
    "Documents": [".pdf", ".docx", ".txt", ".xlsx", ".pptx"],
    "Archives": [".zip", ".rar", ".7z", ".tar", ".gz"],
    "Code": [".py", ".js", ".html", ".css", ".java", ".cpp"],
}

# =========================
# LOGGING
# =========================

logging.basicConfig(
    filename="organizer.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

# =========================
# HASHING
# =========================

def get_file_hash(filepath):
    md5 = hashlib.md5()

    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            md5.update(chunk)

    return md5.hexdigest()

# =========================
# DUPLICATES
# =========================

def find_duplicates(files):
    hashes = {}
    duplicates = []

    for file in files:
        try:
            file_hash = get_file_hash(file)

            if file_hash in hashes:
                duplicates.append(file)
            else:
                hashes[file_hash] = file

        except Exception as e:
            logging.error(f"Hash error: {file} -> {e}")

    return duplicates

# =========================
# CATEGORIES
# =========================

def get_category(extension):
    extension = extension.lower()

    for category, extensions in FILE_TYPES.items():
        if extension in extensions:
            return category

    return "Others"

# =========================
# ORGANIZER
# =========================

def organize_directory(directory):

    directory = Path(directory)

    if not directory.exists():
        print("Directory does not exist.")
        return

    files = [
        file
        for file in directory.iterdir()
        if file.is_file()
    ]

    duplicates = find_duplicates(files)

    if duplicates:
        print("\nDuplicate files detected:")
        for file in duplicates:
            print(f"  {file.name}")

    moved_count = 0

    for file in files:

        if file in duplicates:
            continue

        category = get_category(file.suffix)

        target_folder = directory / category
        target_folder.mkdir(exist_ok=True)

        destination = target_folder / file.name

        try:
            shutil.move(str(file), str(destination))

            moved_count += 1

            logging.info(
                f"{file.name} -> {category}"
            )

            print(
                f"Moved: {file.name} -> {category}"
            )

        except Exception as e:
            logging.error(
                f"Move failed: {file.name} -> {e}"
            )

    print("\n===================")
    print(f"Files moved: {moved_count}")
    print(f"Duplicates: {len(duplicates)}")
    print("===================")

# =========================
# MAIN
# =========================

if __name__ == "__main__":

    print("=" * 40)
    print("Smart File Organizer")
    print("=" * 40)

    folder = input(
        "\nEnter folder path: "
    ).strip()

    organize_directory(folder)
