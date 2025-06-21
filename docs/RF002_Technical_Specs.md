# RF002 ReportFormatter Pro 技術仕様書【ローカル環境・詳細実装】

## 📋 技術仕様概要

| 項目 | 仕様 |
|------|------|
| **アーキテクチャ** | Node.js + Express + Puppeteer |
| **実行環境** | ローカル環境（localhost:3000） |
| **PDF生成エンジン** | Puppeteer (Chrome品質) |
| **UI** | HTML5 + CSS3 + Vanilla JavaScript |
| **文書解析** | mammoth.js + pdf-parse + 独自解析エンジン |
| **配布形態** | 実行ファイル化（pkg使用） |

---

## 🏗️ システムアーキテクチャ

### ディレクトリ構造
```
rf002-local/
├── src/
│   ├── app.js                 // Express メインサーバー
│   ├── routes/
│   │   ├── upload.js          // ファイルアップロード処理
│   │   ├── process.js         // 文書処理API
│   │   └── generate.js        // PDF生成API
│   ├── engines/
│   │   ├── DocumentParser.js  // 文書解析エンジン【詳細実装】
│   │   ├── LayoutEngine.js    // レイアウト処理エンジン
│   │   └── PdfGenerator.js    // PDF生成エンジン【詳細実装】
│   ├── templates/
│   │   ├── mckinsey.html      // McKinseyスタイル
│   │   ├── executive.html     // Executiveスタイル
│   │   └── bcg.html           // BCGスタイル
│   └── utils/
│       ├── config.js          // 設定管理
│       └── logger.js          // ログ管理
├── public/
│   ├── index.html             // メインUI
│   ├── css/
│   │   └── style.css          // UIスタイル
│   ├── js/
│   │   └── app.js             // フロントエンドロジック
│   └── assets/                // フォント・画像
├── output/                    // 生成PDF保存先
├── temp/                      // 一時ファイル保存
├── package.json
└── README.md
```

---

## 🔧 核心エンジン詳細実装

### 1. DocumentParser.js【具体的実装仕様】

#### Word (.docx) 解析の詳細実装
```javascript
const mammoth = require('mammoth');
const AdmZip = require('adm-zip');

class WordDocumentParser {
  async parseWordDocument(filePath) {
    try {
      // Step 1: mammoth.jsでHTMLとスタイル情報を抽出
      const result = await mammoth.convertToHtml({
        path: filePath,
        options: {
          styleMap: [
            "p[style-name='Heading 1'] => h1",
            "p[style-name='Heading 2'] => h2",
            "p[style-name='Heading 3'] => h3",
            "p[style-name='List Paragraph'] => li"
          ]
        }
      });
      
      // Step 2: 詳細な構造情報取得のためXML直接解析
      const zip = new AdmZip(filePath);
      const documentXml = zip.readAsText('word/document.xml');
      const stylesXml = zip.readAsText('word/styles.xml');
      
      // Step 3: 構造解析（見出し階層・リスト構造）
      const structure = this.analyzeDocumentStructure(documentXml);
      
      // Step 4: 統合データ作成
      return {
        html: result.value,
        structure: structure,
        metadata: this.extractMetadata(documentXml),
        styles: this.analyzeStyles(stylesXml)
      };
      
    } catch (error) {
      throw new Error(`Word文書解析エラー: ${error.message}`);
    }
  }
}
```

### 2. PdfGenerator.js【Puppeteer高品質PDF生成】

```javascript
const puppeteer = require('puppeteer');

class PdfGenerator {
  async generateHighQualityPdf(html, template, customOptions = {}) {
    const browser = await puppeteer.launch({
      headless: true,
      args: [
        '--no-sandbox',
        '--disable-setuid-sandbox',
        '--font-render-hinting=none',  // フォント品質向上
        '--disable-font-subpixel-positioning'
      ]
    });
    
    const page = await browser.newPage();
    
    try {
      await page.setContent(html);
      
      const pdf = await page.pdf({
        format: 'A4',
        printBackground: true,
        preferCSSPageSize: true,
        margin: { top: '0', right: '0', bottom: '0', left: '0' }
      });
      
      return pdf;
    } finally {
      await page.close();
      await browser.close();
    }
  }
}
```

---

## 📊 性能・品質仕様

### 処理性能目標
- **文書解析時間**: 30,000文字で10秒以内
- **PDF生成時間**: A4・10ページで20秒以内  
- **メモリ使用量**: 最大1GB
- **ファイルサイズ**: 生成PDF 5MB以内

### 品質保証
- **フォント品質**: サブピクセルレンダリング対応
- **レイアウト精度**: デザイナー品質レベル
- **色彩再現**: sRGB色空間準拠
- **PDF準拠**: PDF/A-1b対応

---

## 🎯 実装完了基準

### 技術的完了基準
1. **全API動作確認**: 200レスポンス・期待データ返却
2. **PDF品質確認**: プロフェッショナルレベル達成  
3. **エラー処理確認**: 全エラーケース適切処理
4. **性能基準達成**: 全処理時間目標値内
5. **実行ファイル化**: スタンドアロン動作確認

---

**作成者**: 立法権（Claude Chat）  
**審査対象**: 司法権（Gemini）  
**実装担当**: 行政権（Claude Code）  
**作成日**: 2025-06-21  
**バージョン**: 1.0（初版）