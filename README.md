# cicd

Flutter project to explain CI/CD.

## Getting Started

This project is a starting point for a Flutter application.

# Flutter CI/CD Pipeline with GitHub Actions

This document explains how to set up a **Continuous Integration and Continuous Deployment (CI/CD)** pipeline for a **Flutter** project using **GitHub Actions**. 
The pipeline automates testing, building, and deploying the app for **Android, iOS, and Web**.

---

## **1. Prerequisites**
- A **Flutter project** hosted on **GitHub**.
- **GitHub Actions** enabled for the repository.
- (Optional) **Firebase/App Store/Play Store** credentials if deploying.

---

## **2. CI/CD Workflow Overview**
The pipeline includes the following stages:
1. **Code Checkout** – Fetch the latest code.
2. **Setup Flutter** – Install Flutter SDK and dependencies.
3. **Run Tests** – Execute unit and widget tests.
4. **Build Artifacts** – Generate APK (Android), IPA (iOS), and Web builds.
5. **Deploy (Optional)** – Upload to Firebase Hosting, GitHub Releases, or app stores.

---

## **3. Setup GitHub Actions**
Create `.github/workflows/flutter-ci-cd.yml` with the following:

## **4. Required Secrets & Fastlane Setup**
🔑 GitHub Secrets (Repo Settings → Secrets → Actions)
Secret Name	Description
GOOGLE_PLAY_SERVICE_ACCOUNT	Google Play JSON API key (Fastlane).
APP_STORE_CONNECT_API_KEY	App Store Connect API Key (Fastlane).
FIREBASE_SERVICE_ACCOUNT	Firebase Hosting service account.


## 5. How to use fastlane
🚀 How to Use It?
In your terminal:

For Android (upload to Google Play Internal testing):
fastlane deploy_playstore

For iOS (upload to TestFlight):
fastlane deploy_testflight

```yaml



name: Flutter CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
    test:
        name: Run Tests
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: subosito/flutter-action@v2
            with:
            flutter-version: '3.19.0' # Change if needed
        - run: flutter pub get
        - run: flutter test

    build-android:
        name: Build Android APK
        needs: test
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: subosito/flutter-action@v2
        - run: flutter pub get
        - run: flutter build apk --release
        - uses: actions/upload-artifact@v4
            with:
            name: release-apk
            path: build/app/outputs/flutter-apk/app-release.apk

    build-ios:
        name: Build iOS IPA (Simulator)
        if: github.ref == 'refs/heads/main'
        needs: test
        runs-on: macos-latest
        steps:
        - uses: actions/checkout@v4
        - uses: subosito/flutter-action@v2
        - run: flutter pub get
        - run: flutter build ios --release --no-codesign
        - uses: actions/upload-artifact@v4
            with:
            name: release-ipa
            path: build/ios/iphoneos/Runner.app

    build-web:
        name: Build Web
        needs: test
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: subosito/flutter-action@v2
        - run: flutter pub get
        - run: flutter build web --release
        - uses: actions/upload-artifact@v4
            with:
            name: web-release
            path: build/web

    # Optional: Deploy to Firebase Hosting
    deploy-web:
        name: Deploy to Firebase
        needs: build-web
        if: github.ref == 'refs/heads/main'
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v4
        - uses: subosito/flutter-action@v2
        - run: flutter pub get
        - run: flutter build web --release
        - uses: FirebaseExtended/action-hosting-deploy@v0
            with:
            repoToken: '${{ secrets.GITHUB_TOKEN }}'
            firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
            projectId: your-project-id
            channelId: live

    deploy-ios:  
        name: Deploy to App Store  
        needs: build-ios  
        if: github.ref == 'refs/heads/main'  
        runs-on: macos-latest  
        steps:  
        - uses: actions/checkout@v4  
        - uses: ruby/setup-ruby@v1  
            with:  
            ruby-version: '3.1.2'  
        - run: gem install fastlane -NV  
        - run: fastlane deploy_to_appstore  

    deploy-android:  
        name: Deploy to Google Play Store  
        needs: build-android  
        if: github.ref == 'refs/heads/main'  
        runs-on: ubuntu-latest  
        steps:  
        - uses: actions/checkout@v4  
        - uses: ruby/setup-ruby@v1  
            with:  
            ruby-version: '3.1.2'  
        - run: gem install fastlane -NV  
        - run: fastlane supply --aab $(find . -name "*.aab") --track internal --json_key "${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT }}"  