# Web UI自動化測試規範

[Here is English version](https://github.com/hayasilin/web-ui-automation-tests-guide)

這是總結個人在開發可靠及穩定的Web UI自動化測試所得之規範及最佳實踐，歡迎同好的[反饋](https://github.com/hayasilin/web-ui-automation-tests-guide/issues) 和 [Pull requests](https://github.com/hayasilin/web-ui-automation-tests-guide/pulls)。

如果你是Mobile app的開發者，想了解關於Mobile app的UI自動化測試，可以參考我另外一篇[iOS 及 Android UI自動化測試規範](https://github.com/hayasilin/ios-android-ui-automation-tests-guide/blob/master/README_zh-hant.md)

## 介紹

本規範適合在Web UI自動化測試領域已經有些許經驗的工程師，使用Selenium及WebDriver的測試框架，或使用Cypress測試框架進行開發，且將UI自動化測試整合至團隊的CI/CD流程中。

關於Selenium及WebDriver測試框架，如果這裡沒提到，那就在以下的文件裡：
- Selenium
  - [Selenium.dev](https://selenium.dev/)
  - [Selenium WebDriver](https://selenium.dev/projects/)
  - [Selenium with Python](https://selenium-python.readthedocs.io/index.html)

- Cypress
  - [Cypress.io](https://www.cypress.io/)

## 目錄
- [開始開發UI自動化測試之前](#開始開發UI自動化測試之前)
  - [UI自動化測試開始很簡單但維護很昂貴](#UI自動化測試開始很簡單但維護很昂貴)
  - [什麼是Flaky tests](#什麼是Flaky-tests)
  - [為何UI自動化測試會有Flaky tests](#為何UI自動化測試會有Flaky-tests)
- [讓UI自動化測試可靠及穩定的最佳實踐](#讓UI自動化測試可靠及穩定的最佳實踐)
  - [遵從測試三角形](#遵從測試三角形)
  - [開發前的準備](#開發前的準備)
  - [什麼樣的測試適合UI自動化測試](#什麼樣的測試適合UI自動化測試)
  - [避免Flaky tests的10大實用方法](#避免Flaky-tests的10大實用方法)
- [UI測試的程式範例](#UI測試的程式碼範例)
- [常見問答](#常見問答)

## 開始開發UI自動化測試之前

### UI自動化測試開始很簡單但維護很昂貴
- UI自動化測試看起來很神奇，它模擬使用者的行為並自動測試你的Web，我們需要它但實際上很昂貴。
- 因為Web的UI是不斷變動的，所以測試碼也需要跟著變動，別忘了這些維護成本。
- 早期UI自動化測試是開發者用來驗證更改程式碼後，並沒造成損壞的開發者工具。不過近年來也成為QA團隊的測試工具，希望能用來減少很花時間的手動測試。但很容易會認為我們能自動化所有原本的手動測試，甚至可以達成捨棄手動測試的目標，當你開始這樣做卻忘了UI自動化測試的限制及維護成本，你首先會遇到惡名昭彰的**Flaky tests**。
- Flaky tests會導致不可靠也不穩定的UI自動化測試，造成UI自動化測試無法幫助QA測試：
  - 不穩定的結果只會讓QA及團隊對UI自動化測試失去信心，也不會相信測試後的報告，結果是你的團隊仍然會花時間去執行手動測試，這既花時間也讓你的團隊變慢。
  - 最後沒有人會繼續維護UI自動化測試，因為團隊再也沒人在意。

### 什麼是Flaky tests
- 當執行自動化測試時，有些測試項目這次會成功但下次會失敗，成功及失敗會反覆出現，即便你並沒有更動程式碼，而用人力親自去確認Web其實沒問題。我們無法預測該測項下次會成功還是失敗。
- 下面是Flaky tests造成的例子。測項A跟測項C是Flaky tests因為它們成功及失敗的結果輪流出現，相對的測項B一直成功，是穩定的測項。
- 一般而言我們會把UI自動化測試放入CI/CD流程之中，並根據所設定的情況去自動啟動它。最常見的方式是與Jenkins整合。如果我們測項中有Flaky tests，那麼以下例子顯示每次Jenkins測試完畢後結果都很有可能是紅燈，因為哪怕只要是1個測項失敗，整個Jenkins Job結果就會顯示錯誤，結果是團隊必須要派人力去檢查為何測試失敗。
- 以下例子只有3個測項，如果你的團隊已經埋頭苦幹先寫好了100多個測項，想像一下如果測試沒寫好裡面有Flaky tests，你的團隊人員要花多少時間去確認是否真的是Web的問題，還是Flaky tests造成的。

| Test Runs   | 1st     | 2nd      | 3rd      | 4th      | 5th     | 6th      | 7th      | 8th      | 9th      | 10th     | Next run?              |
| ----------- | ------- |--------- | ---------| -------- | ------- | -------- | -------- | -------- | -------- | -------- | ---------------------- |
| Test Case A | Success | **Fail** | Success  | Success  | Success | **Fail** | Success  | Success  | **Fail** | **Fail** | **?**                  | 
| Test Case B | Success | Success  | Success  | Success  | Success | Success  | Success  | Success  | Success  | Success  | **?**                  | 
| Test Case C | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | Success  | **Fail** | **?**                  | 
| ...                                                                                                                                              |
| Jenkins     | Success | **Fail** | **Fail** | **Fail** | Success | **Fail** | **Fail** | **Fail** | **Fail** | **Fail** | **Highly likely Fail** |

- 為何Flaky tests很糟糕:
  - 團隊會對UI自動化測試失去信心。
  - 如果你的CI/CD設計當UI自動化測試成功後將繼續其他動作，Flaky tests將停滯住CI/CD流程。
  - 無法達成Web快速上線，更別提快速提供反饋讓團隊維持或提升Web品質。

- Google處理Flaky tests的經驗
  - 2015 [Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
  - 2016 [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
  - 2017 [Where do our flaky tests come from](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html)
  
### 為何UI自動化測試會有Flaky tests

- UI自動化測試是測試整個Web在end-to-end的串接是否運作正常，範圍包含了UI顯示，API資料解析及顯示，跨Web行為操作，以及使用者的各種狀態改變等等。以上動作包含著無數種變化可能，只要有1個部分失敗，整個測試會失敗，此種特性也讓UI自動化測試本質上較為脆弱。
- 開發測試時，最重要的觀念是**測試孤立化**，如果沒做好則測試結果往往就會受到各種依賴的影響。
- 測試框架並不是萬能，它有其限制及本身的問題，在某些狀況下會造成測試失敗，而原因並不是你寫的程式碼。

**團隊內部溝通造成的Flaky因子:**
- UI及流程在每次改版都可能會改變。
- 不穩定的測試環境及測試資料。
- 元件的產生時間會根據框架版本及API回傳處理時間而變動，有時很快就產生但有時會花很長時間，不要使用等待(Sleep)，請使用等待元件顯示後才對元件執行動作的方法(Wait until)。
- 動態變化的UI元件: UI會根據伺服器端的設定而更改UI。
- 寫得不好的測試程式碼。

**外部因素造成的Flaky因子:**
- 測試框架已知的問題(Selenium WebDriver / Cypress)。
- 各種瀏覽器行為的不同（感謝老天現今可以以Chrome測試為主即可，過去需要測試IE/Chrome/Firefox的時代簡直是災難）。
- 跨Web測試的不穩定。
- 平行化測試的不穩定。
- 透過IDE驗證測試及透過Command line工具執行測試，兩者有時會有些許不同的行為。
- 以及 **更多**。

## 讓UI自動化測試可靠及穩定的最佳實踐

### 遵從測試三角形

根據Google的[Just Say No to More End-to-End Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)這篇文章，測試三角形是想像有個三角形，底部面積最大的部分是你的單元測試，接著往上你的測試會變得越來越大，但同時測試的數量會變得越來越少。

Google建議70/20/10的比例：70%單元測試，20%整合測試，以及10%的UI測試。實際的比例會根據每個團隊而有所不同，但整體來說，比例應該如同三角形的形狀一樣。另外請避免反模式，比如團隊主要依賴UI測試，並很少使用整合測試，且幾乎沒有單元測試。

如同正常的三角形在現實世界是最穩定的結構，測試三角形也是最穩定的測試策略。

### 開發前的準備
- 了解到維護成本，不同裝置依賴問題，測試環境的依賴問題，原生測試框架的限制等造成UI自動化測試的脆弱性。
- 因此，UI自動化測試應只是執行**煙霧測試(Smoke Testing)**，確認最常用的使用者情境即可，此外不需要做更多。
- 如果發現有某隻測項容易產生Flaky結果但無法快速解決，請移除或不執行該測項，Flaky tests不該被留存，直到被修復。
- 團隊沒有時間一直檢查Flaky tests，只有可靠且穩定的UI自動化測試可以幫助團隊：
  - 快速交付Web。
  - 快速反饋，以即時維護及強化Web品質。

### 什麼樣的測試適合UI自動化測試

**推薦**
- 成熟的功能：產品最重要的功能且短期內不會輕易被改變。
- 最常見的使用者流程。
- 很簡單就能被轉換成UI自動化測試的功能。

**不推薦**
- 仍在發展中的功能：該功能可能依照商業計劃仍會不斷變動。
- 因測試框架的限制，所以很難被變成自動化測試的測項。
- 需要花費很多成本才能做的測項。

**UI自動化測試無法將所有的測項都變成自動化，請選擇適合的測項並讓不適合的測項仍然以手動測試為主。**

### 避免Flaky tests的10大實用方法
1. 測試孤立：測試應該要獨立於其他測試之外。如果有測項改變狀態或調整資料庫，請在測項的setup或teardown清空該狀態。
2. 確認所有UI會產生的狀態。寫測項時當下所看到的UI並不代表所有狀態下都是該UI，當使用者的狀態改變，UI可能也會隨之變化，將所有可能的UI情境都思考過並讓測試能充分對應各種情境才能避免Flaky。
3. 每次完成測試程式碼後，請先將所有測項一起執行，並重複約10次的測試，確保新加入的程式碼不會造成Flaky tests。
4. 完成測試程式碼後，試著先在你定期執行UI自動化測試的機器上執行，確保沒有額外的問題。
5. 將測項所需的畫面跳轉減至最少。讓測試開始時從所需測試的畫面(URL)開始即可，既可加快測試速度，也可避免Flaky tests。
6. 選擇Web最成熟的功能開始寫測試程式。
7. 不要過度設計你的測試程式，使用設計模式如Page object pattern讓每個測試程式既簡單又很容易維護。
8. 穩定度重要性大於完成度，不要想要寫越多測試越好，只需要測試最常用的功能，別忘了UI自動化測試只是煙霧測試(Smoke Testing)。不穩定的UI自動化測試對於團隊毫無幫助，因為需要浪費時間去調查Flaky tests而你的Web實際上功能並無問題。
9. 每個測試項目只挑選畫面上1個最重要的UI元件驗證(Assert)即可。UI測試這個名稱常常讓人誤以為要測試整個畫面的所有UI都正確，但這只會增加Flaky tests的機會，實務上請把UI測試想成測試使用者的行為或情境，而不是用來測試UI位置或佈局。
10. 不需使用太多的瀏覽器版本執行平行化測試，數個瀏覽器版本的測試結果應該就提供團隊足夠的信心程度來決定Web可否上線。

## UI測試的程式碼規範

**開啟瀏覽器**

```java
WebDriver driver = new ChromeDriver();
```

**開啟特定網址**

```java
driver.get("http://www.google.com");
```

**透過name attribute找到element**

```java
WebElement element = driver.findElement(By.name("name"));
```

**在input中輸入字串**

```java
element.sendKeys("text");
```

**Submit the form**

```java
element.submit();
```

**檢查此頁的Page title**

```java
driver.getTitle();
```

**等待10秒讓頁面執行完成**

```java
(new WebDriverWait(driver, 10)).until(new ExpectedCondition<Boolean>()
{
    public Boolean apply(WebDriver d)
    {
        return driver.getTitle().toLowerCase().startsWith("text");
    }
});

```

**關閉瀏覽器**

```java
driver.quit();
```


## 常見問答
1. 在CI/CD流程中，多久需執行一次UI自動化測試確認Web正常運作？
  - **建議1天1次即可**，在凌晨或半夜。
    - 因為團隊裡不論是客戶端還是伺服器端的工程師，往往需在工作時間時更新程式碼或更新環境，此時執行UI自動化測試只是讓Flaky tests發生的機會大增。除非你的團隊能確保一個超級穩定的測試環境及測試資料（但又是另1種成本），不然其實只需要在1天中選1個或2個最不會被影響的時間執行UI自動化測試確認Web沒問題即可，而結果也足以提供團隊即時的反饋。
    - 理論中，會希望將UI自動化測試啟動時間設定如同單元測試一樣，希望能在每次工程師發Pull Request上傳程式碼時，不僅要執行單元測試，也連同UI測試一起執行，來確認所有測試都能通過後，才能將程式碼Merge進分支。但實務上很難做到，主要原因是當你的UI測試項目越來越多，執行所需的時間也越來越長，如同大家所知，單元測試時間短，但UI測試時間長，如果把UI測試也排進去會拉長工程師及該程式碼等待的時間，造成效率降低的問題。此外，如同前面所述，UI自動化測試在本質上是脆弱的，常會因各種不同的因素而造成失敗的結果，如果UI測試不斷有問題，也會造成阻斷CI/CD後續流程的問題，所以實務上建議不需要將UI自動化測試與Pull request的流程綁在一起。

2. 使用哪種測試框架及工具比較好？
  - Selenium WebDriver仍是主流，不過後起之秀如Cypress也推薦使用。
    - Selenium WebDriver可說是Web的UI自動化測試的代名詞，歷史悠久且被廣泛使用，如果你已經在使用它，現在仍是個不錯的工具。
    - 後起之秀如Cypress提供了除Selenium WebDriver外的選擇。與Selenium WebDriver不同，Cypress建構在Mocha及Chai工具之上，並透過將Chrome封裝起來方式，提供了更加原生的方式來測試Web，對Web的掌控性也更強，如果你才剛入門Web的UI自動化測試，推薦可以使用Cypress。