
```
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


```

Reference
```
[ ] Apexテスト
https://base.terrasky.co.jp/articles/sw87f

```

helloLWC.test.js
```
import { createElement } from "lwc";
import HelloLWC from "c/helloLWC";
import selectAccounts from "@salesforce/apex/AccountController.search";

// Apexをモック
jest.mock(
  "@salesforce/apex/AccountController.search",
  () => {
    return {
      default: jest.fn()
    };
  },
  { virtual: true }
);

describe("c-hello-lwc", () => {
  afterEach(() => {
    while (document.body.firstChild) {
      document.body.removeChild(document.body.firstChild);
    }
    jest.clearAllMocks();
  });

  it("shows error message on Apex failure", async () => {
    // 失敗をモック
    selectAccounts.mockRejectedValue(new Error("Apex error"));

    const element = createElement("c-hello-lwc", { is: HelloLWC });
    document.body.appendChild(element);

    // 十分に待つ：最低3回
    await Promise.resolve(); // 1回目: catch実行
    await Promise.resolve(); // 2回目: state変更
    await Promise.resolve(); // 3回目: DOM更新完了

    const error = element.shadowRoot.querySelector(".slds-text-color_error");
    expect(error).not.toBeNull();
    expect(error.textContent).toBe("アカウント取得に失敗しました");
  });
});

```

helloLWC.html
```
<template>
  <article class="slds-card">
    <div class="slds-card__body slds-card__body_inner">
      <div>Hello LWC!</div>

      <template if:true={error}>
        <div class="slds-text-color_error">{error}</div>
      </template>

      <template for:each={accounts} for:item="acc">
        <div key={acc.Id}>{acc.Name}</div>
      </template>
    </div>
  </article>
</template>
```

helloLWC.js

```
import { LightningElement } from "lwc";

import selectAccounts from "@salesforce/apex/AccountController.search";

export default class HelloLWC extends LightningElement {
  accounts = [];
  error = null;

  connectedCallback() {
    selectAccounts()
      .then((res) => {
        this.accounts = res;
      })
      .catch(() => {
        this.error = "アカウント取得に失敗しました";
      });
  }
}
```

helloLWC.js-meta.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>63.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
    </targets>
</LightningComponentBundle>
```

AccountControllerTest.cls
```
@isTest
private class AccountControllerTest {

    // テストで使う共通データを1回だけセットアップ
    @testSetup
    static void setupData() {
        insert new List<Account>{
            new Account(Name = 'Test Account A'),
            new Account(Name = 'Test Account B')
        };
    }

    @isTest
    static void testSearchReturnsAccounts() {
        // 実行＆検証
        Test.startTest();
        List<Account> results = AccountController.search();
        Test.stopTest();

        // 検証
        System.assertEquals(2, results.size(), '2件のアカウントが返るべき');
        System.assertNotEquals(null, results[0].Id, 'IDは取得されているべき');
        System.assertNotEquals(null, results[0].Name, 'Nameは取得されているべき');
    }

    @isTest
    static void testSearchHandlesExceptionGracefully() {
        // AccountController.search()が例外をthrowする状況を作るにはモックが必要ですが、
        // 今回の例ではAuraHandledExceptionがcatchされるパスに入るのは難しい。
        // 実際には、リファクタリングして「検索対象を引数で受け取る」ようにしてテスト可能性を高めるのが現実解。

        // 例外網羅を意識して、カバレッジ目的だけで空呼びする例（参考）
        Test.startTest();
        try {
            AccountController.search(); // 実際は成功する
        } catch (Exception ex) {
            System.assert(false, '例外は発生すべきでない');
        }
        Test.stopTest();
    }
}

```

AccountControllerTest.cls-meta.xlm
```
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>63.0</apiVersion>
    <status>Active</status>
</ApexClass>
```

AccountController.cls
```
{
    @AuraEnabled
    public static List<Account> search() {
        try
        {
            return [
                SELECT Id, Name FROM Account
            ];
        }
        catch (Exception e)
        {
            throw new AuraHandledException(e.getMessage());
        }
    }
}
```

AccountController.cls-meta.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<ApexClass xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>63.0</apiVersion>
    <status>Active</status>
</ApexClass>
```

