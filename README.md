name: Build Android APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y git zip unzip openjdk-17-jdk python3-pip python3-venv

      - name: Unpack project
        run: |
          mkdir -p project

          ZIP_FILE=$(find . -maxdepth 3 -type f -iname '*.zip' | head -n 1)

          if [ -z "$ZIP_FILE" ]; then
            echo "No ZIP project file found"
            exit 1
          fi

          echo "Using ZIP: $ZIP_FILE"

          unzip -o "$ZIP_FILE" -d project

          SPEC=$(find project -name buildozer.spec -type f | head -n 1)

          if [ -z "$SPEC" ]; then
            echo "buildozer.spec not found"
            exit 1
          fi

          PROJECT_DIR=$(dirname "$SPEC")

          echo "PROJECT_DIR=$PROJECT_DIR"
          echo "PROJECT_DIR=$PROJECT_DIR" >> "$GITHUB_ENV"

      - name: Install Buildozer
        run: |
          python3 -m venv ~/venv
          source ~/venv/bin/activate
          pip install --upgrade pip setuptools wheel
          pip install buildozer cython

      - name: Build APK
        run: |
          source ~/venv/bin/activate
          cd "$PROJECT_DIR"
          buildozer android debug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: android-apk
          path: ${{ env.PROJECT_DIR }}/bin/*.apk
          if-no-files-found: error
