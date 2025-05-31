
【手順】
### Developer Edition環境を新規作成してログイン
### 設定 → 開発 → DevHubを有効化
### Lightning Design System → UIコンポーネント

SFDX: Create Project
    └ Type
    └ Standard
  └ Name
    └ Free
  └ SaveDir
    └ Free
SFDX: Authorize a DevHub
  └ Name
    └ Free
SFDX: Create a Default Scratcch Org
  └ Type
    └ Default
  └ Name
    └ Free
SFDX: Open Default Scratch Org
SFDX: Create Lightning Web Component
  └ Name
    └ Free
SFDX: Create Apex Class
sfdx force:lightning:lwc:test:setup
  └ 手動で実行する場合
    └ npm install --save-dev @salesforce/sfdx-lwc-jest
    └ 上記実行後に Package.json の scripts に下記を追加
    └　"scripts": {
       "test:unit": "sfdx-lwc-jest"
       }
SFDX: Run Apex Tests
npm run test:unit
SFDX: Push Source to Default Org
SFDX: Deploy This Source To Org

ExampleController.cls
ExampleController.cls-meta.xml
  └ apiVersion
    └ 63.0
  └ status
    └ Active
ExampleTest.cls
  └ private class && static void
  └ @isTest
  └ @testSetup
ExampleTest.cls-meta.xml
  └ apiVersion
    └ 63.0
  └ status
    └ Active
Example.test.js
Example.html
Example.js
Example.js-meta.xml
    └ isExposed
        └ true
    └ targets
        └ target
            └ lightning__AppPage

