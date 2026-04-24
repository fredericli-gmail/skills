# 簡報 Skill — python-pptx 實作配方

> 本檔提供 `python-pptx` 的具體操作範例，對應 [SKILL.md](SKILL.md) 中的版面規範。
> 所有範例皆以 **960×540pt（16:9）** 為前提，使用 **Microsoft JhengHei** 字型。

---

## 0. 安裝與匯入

```bash
pip install python-pptx Pillow
# 截圖驗證需要 LibreOffice（macOS：brew install --cask libreoffice）
```

```python
# 匯入 python-pptx 核心模組
from pptx import Presentation
# 匯入尺寸與字級單位（Pt 可同時用於座標與字級）
from pptx.util import Pt, Emu
# 匯入顏色類別
from pptx.dml.color import RGBColor
# 匯入字型對齊、形狀類型
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
```

---

## 1. 規範常數（集中管理）

```python
# ============ 版面尺寸 ============
# 投影片寬度 960pt
SLIDE_W = Pt(960)
# 投影片高度 540pt
SLIDE_H = Pt(540)

# ============ 字型 ============
# 全簡報統一字型
FONT = "Microsoft JhengHei"

# ============ 文字色 ============
# 標題深色
C_TITLE = RGBColor(0x0F, 0x1B, 0x3D)
# 副標 / 描述
C_SUB = RGBColor(0x64, 0x74, 0x8B)
# 卡片內文
C_BODY = RGBColor(0x1E, 0x29, 0x3B)
# 頁尾文字
C_FOOTER = RGBColor(0x94, 0xA3, 0xB8)
# 白色（深色背景上的文字）
C_WHITE = RGBColor(0xFF, 0xFF, 0xFF)

# ============ 強調色 ============
# 主強調 橘
C_ORANGE = RGBColor(0xEA, 0x58, 0x0C)
# 綠
C_GREEN = RGBColor(0x05, 0x96, 0x69)
# 藍綠
C_TEAL = RGBColor(0x08, 0x91, 0xC2)
# 藍
C_BLUE = RGBColor(0x25, 0x63, 0xEB)
# 紫
C_PURPLE = RGBColor(0x7C, 0x3A, 0xED)
# 琥珀
C_AMBER = RGBColor(0xF5, 0x9E, 0x0B)

# ============ 背景與裝飾 ============
# 分隔線
C_LINE = RGBColor(0xE2, 0xE8, 0xF0)
# 卡片底色（淡橘）
C_CARD_ORANGE = RGBColor(0xFF, 0xF7, 0xED)
# 卡片底色（淡藍）
C_CARD_BLUE = RGBColor(0xEF, 0xF6, 0xFF)
# 卡片底色（淡綠）
C_CARD_GREEN = RGBColor(0xEC, 0xFD, 0xF5)
# 卡片底色（淡紫）
C_CARD_PURPLE = RGBColor(0xFA, 0xF5, 0xFF)
```

---

## 2. 建立 16:9 簡報骨架

```python
def create_deck():
    # 建立空白簡報
    prs = Presentation()
    # 設定投影片寬度（960pt）
    prs.slide_width = SLIDE_W
    # 設定投影片高度（540pt）
    prs.slide_height = SLIDE_H
    # 回傳簡報物件
    return prs
```

---

## 3. 字型套用輔助函式

```python
def apply_font(run, size_pt, bold=False, color=C_TITLE):
    # 設定字型名稱
    run.font.name = FONT
    # 設定字級（Pt）
    run.font.size = Pt(size_pt)
    # 設定是否粗體
    run.font.bold = bold
    # 設定文字顏色
    run.font.color.rgb = color


def add_text(slide, left, top, width, height, text, size_pt,
             bold=False, color=C_TITLE, align=PP_ALIGN.LEFT,
             line_spacing=1.0):
    # 在指定位置新增文字方塊
    box = slide.shapes.add_textbox(Pt(left), Pt(top), Pt(width), Pt(height))
    # 取得文字框
    tf = box.text_frame
    # 關閉自動調整大小（避免 python-pptx 自動改變字級破壞版面）
    tf.word_wrap = True
    # 取第一段落
    p = tf.paragraphs[0]
    # 設定對齊方式
    p.alignment = align
    # 設定行距
    p.line_spacing = line_spacing
    # 新增 run 並寫入文字
    run = p.add_run()
    run.text = text
    # 套用字型
    apply_font(run, size_pt, bold=bold, color=color)
    # 回傳文字方塊物件（供後續調整位置）
    return box
```

---

## 4. 頂部色帶 + 圓形圖標 + 標題區

```python
def add_header(slide, page_no, total, title, subtitle, accent=C_ORANGE):
    # ① 頂部色帶：全寬 × 9pt 高、top=0
    bar = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Pt(0), Pt(0), SLIDE_W, Pt(9)
    )
    # 填色為主強調色
    bar.fill.solid()
    bar.fill.fore_color.rgb = accent
    # 移除外框線
    bar.line.fill.background()

    # ② 圓形圖標：left=36, top=30, 40×40pt（與色帶間距 = 30-9 = 21pt，滿足 ≥ 20pt）
    oval = slide.shapes.add_shape(
        MSO_SHAPE.OVAL, Pt(36), Pt(30), Pt(40), Pt(40)
    )
    oval.fill.solid()
    oval.fill.fore_color.rgb = accent
    oval.line.fill.background()
    # 圓內頁碼文字
    tf = oval.text_frame
    # 關閉內距，確保數字置中
    tf.margin_left = tf.margin_right = tf.margin_top = tf.margin_bottom = 0
    p = tf.paragraphs[0]
    p.alignment = PP_ALIGN.CENTER
    run = p.add_run()
    # 頁碼用兩位數（01, 02...）
    run.text = f"{page_no:02d}"
    apply_font(run, 14, bold=True, color=C_WHITE)

    # ③ 標題：left=86, top=33, 28pt 粗體
    add_text(slide, 86, 33, 800, 36, title, 28, bold=True, color=C_TITLE)

    # ④ 副標：left=86, top=64, 14pt
    add_text(slide, 86, 64, 800, 24, subtitle, 14, color=C_SUB)

    # ⑤ 分隔線：left=36, top=98, 寬 888pt, 高 0.5pt
    line = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Pt(36), Pt(98), Pt(888), Pt(0.5)
    )
    line.fill.solid()
    line.fill.fore_color.rgb = C_LINE
    line.line.fill.background()
```

---

## 5. 頁尾（左側機密標註 + 右側頁碼 + 底部裝飾線）

```python
def add_footer(slide, page_no, total):
    # 底部裝飾線：left=36, top=510, 寬 888pt
    deco = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Pt(36), Pt(510), Pt(888), Pt(0.5)
    )
    deco.fill.solid()
    deco.fill.fore_color.rgb = C_LINE
    deco.line.fill.background()

    # 左下機密標註
    add_text(
        slide, 36, 515, 600, 16,
        "營運秘密 嚴勿外流（安全等級 - 機密）",
        10, color=C_FOOTER
    )

    # 右下頁碼：XX / 28
    add_text(
        slide, 850, 515, 80, 16,
        f"{page_no:02d} / {total:02d}",
        10, color=C_FOOTER, align=PP_ALIGN.RIGHT
    )
```

---

## 6. 卡片（頂部窄色條 + 標題行 + 描述）

```python
def add_card(slide, left, top, width, height,
             accent, bg, title, desc):
    # ① 卡片底：白/淡色矩形
    card = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Pt(left), Pt(top), Pt(width), Pt(height)
    )
    card.fill.solid()
    card.fill.fore_color.rgb = bg
    # 淡色邊框
    card.line.color.rgb = C_LINE
    card.line.width = Pt(0.5)

    # ② 頂部窄色條：6pt 高
    top_bar = slide.shapes.add_shape(
        MSO_SHAPE.RECTANGLE, Pt(left), Pt(top), Pt(width), Pt(6)
    )
    top_bar.fill.solid()
    top_bar.fill.fore_color.rgb = accent
    top_bar.line.fill.background()

    # ③ 卡片標題（粗體，深色）
    add_text(
        slide, left + 16, top + 16, width - 32, 24,
        title, 14.4, bold=True, color=C_BODY
    )

    # ④ 卡片描述（一般字重）
    add_text(
        slide, left + 16, top + 44, width - 32, height - 56,
        desc, 14.4, color=C_BODY, line_spacing=1.1
    )
```

### 範例：3 欄卡片佈局

```python
def add_three_column_cards(slide, items):
    # 起始 y 在分隔線下方（111pt）
    y = 130
    # 左右邊距 36pt，卡片間距 16pt
    total_w = 960 - 36 * 2
    gap = 16
    card_w = (total_w - gap * 2) / 3
    card_h = 160
    # 三種強調色與對應卡片底色
    palette = [
        (C_ORANGE, C_CARD_ORANGE),
        (C_BLUE, C_CARD_BLUE),
        (C_GREEN, C_CARD_GREEN),
    ]
    # 逐一放置三張卡
    for i, (title, desc) in enumerate(items[:3]):
        accent, bg = palette[i]
        left = 36 + i * (card_w + gap)
        add_card(slide, left, y, card_w, card_h, accent, bg, title, desc)
```

---

## 7. 封面多色 TopBar（6 段各 160pt 寬）

```python
def add_cover_bars(slide):
    # 6 段配色：橘、琥珀、綠、藍綠、藍、紫
    colors = [C_ORANGE, C_AMBER, C_GREEN, C_TEAL, C_BLUE, C_PURPLE]
    # 頂部色帶
    for i, color in enumerate(colors):
        seg = slide.shapes.add_shape(
            MSO_SHAPE.RECTANGLE, Pt(i * 160), Pt(0), Pt(160), Pt(9)
        )
        seg.fill.solid()
        seg.fill.fore_color.rgb = color
        seg.line.fill.background()
    # 底部色帶（順序相反）
    for i, color in enumerate(reversed(colors)):
        seg = slide.shapes.add_shape(
            MSO_SHAPE.RECTANGLE, Pt(i * 160), Pt(531), Pt(160), Pt(9)
        )
        seg.fill.solid()
        seg.fill.fore_color.rgb = color
        seg.line.fill.background()
```

---

## 8. 新增內頁的標準組合

```python
def add_content_slide(prs, page_no, total, title, subtitle, body_fn):
    # 使用空白版型
    blank_layout = prs.slide_layouts[6]
    slide = prs.slides.add_slide(blank_layout)
    # ① 標題區
    add_header(slide, page_no, total, title, subtitle)
    # ② 內容區（由呼叫端注入）
    body_fn(slide)
    # ③ 頁尾
    add_footer(slide, page_no, total)
    return slide
```

---

## 9. 截圖驗證（verify_slide_visual）

`python-pptx` 本身無法輸出圖片，需透過 LibreOffice 轉 PDF → Pillow / pdf2image 取圖。

```python
import subprocess
from pathlib import Path

def verify_slide_visual(pptx_path, slide_index, out_png):
    # ① 先用 LibreOffice 將 pptx 轉成 PDF
    pdf_dir = Path(out_png).parent
    subprocess.run([
        "soffice", "--headless",
        "--convert-to", "pdf",
        "--outdir", str(pdf_dir),
        str(pptx_path)
    ], check=True)
    pdf_path = pdf_dir / (Path(pptx_path).stem + ".pdf")

    # ② 再用 pdf2image 取出指定頁為 PNG
    from pdf2image import convert_from_path
    # DPI 150 足以肉眼檢視溢出 / 重疊
    images = convert_from_path(str(pdf_path), dpi=150,
                               first_page=slide_index + 1,
                               last_page=slide_index + 1)
    images[0].save(out_png, "PNG")
    return out_png
```

> 檢視截圖時，人工（或由 Claude 多模態讀圖）逐項比對 SKILL.md 的三大品質規則。

---

## 10. 元素掃描（verify_slides）— 文字溢出 / 重疊偵測

```python
def scan_slide(slide, slide_index):
    # 收集所有 shape 的座標、尺寸、文字長度
    items = []
    for shp in slide.shapes:
        info = {
            "slide": slide_index,
            "name": shp.name,
            "left": shp.left, "top": shp.top,
            "width": shp.width, "height": shp.height,
            "has_text": shp.has_text_frame,
            "text": shp.text_frame.text if shp.has_text_frame else "",
        }
        items.append(info)
    return items


def find_overflow(items):
    # 以字元寬度 / 行高估算文字實際佔用大小
    # 粗估：14.4pt 的中文字寬 ≈ 14.4pt、行高 ≈ 14.4 * 1.1
    alerts = []
    for it in items:
        if not it["has_text"] or not it["text"]:
            continue
        # 估算每行最大字元數
        est_char_w = 14
        max_chars_per_line = max(1, int(Emu(it["width"]).pt / est_char_w))
        lines = sum(
            max(1, (len(line) + max_chars_per_line - 1) // max_chars_per_line)
            for line in it["text"].splitlines() or [""]
        )
        est_h = lines * 14.4 * 1.1
        actual_h = Emu(it["height"]).pt
        if est_h > actual_h + 1:
            alerts.append({
                "slide": it["slide"], "type": "overflow",
                "name": it["name"],
                "msg": f"預估 {est_h:.1f}pt > 容器 {actual_h:.1f}pt"
            })
    return alerts


def find_overlap(items):
    # 兩兩比對 bounding box
    alerts = []
    text_items = [i for i in items if i["has_text"] and i["text"].strip()]
    for i in range(len(text_items)):
        for j in range(i + 1, len(text_items)):
            a, b = text_items[i], text_items[j]
            # 計算 AABB 交集
            ax2 = a["left"] + a["width"]
            ay2 = a["top"] + a["height"]
            bx2 = b["left"] + b["width"]
            by2 = b["top"] + b["height"]
            ix = max(0, min(ax2, bx2) - max(a["left"], b["left"]))
            iy = max(0, min(ay2, by2) - max(a["top"], b["top"]))
            if ix > 0 and iy > 0:
                alerts.append({
                    "slide": a["slide"], "type": "overlap",
                    "msg": f"{a['name']} 與 {b['name']} 重疊 {Emu(ix).pt:.1f}×{Emu(iy).pt:.1f}pt"
                })
    return alerts


def check_header_spacing(items):
    # 找出第一條頂部色帶（height < 15pt 且 top < Pt(2)）與第一個 Oval（left < Pt(50)）
    alerts = []
    bars = [i for i in items if i["height"] < Pt(15) and i["top"] < Pt(2)]
    ovals = [i for i in items if "Oval" in i["name"] and i["left"] < Pt(50)]
    if bars and ovals:
        gap = Emu(ovals[0]["top"] - (bars[0]["top"] + bars[0]["height"])).pt
        if gap < 20:
            alerts.append({
                "slide": items[0]["slide"], "type": "header_gap",
                "msg": f"色帶 → 圓形圖標 間距 {gap:.1f}pt（< 20pt），需整組下移 {20 - gap:.1f}pt"
            })
    return alerts


def verify_slides(pptx_path):
    # 一次掃描所有頁面
    prs = Presentation(pptx_path)
    all_alerts = []
    for idx, slide in enumerate(prs.slides):
        items = scan_slide(slide, idx)
        all_alerts += find_overflow(items)
        all_alerts += find_overlap(items)
        all_alerts += check_header_spacing(items)
    return all_alerts
```

---

## 11. 常見修復動作（對應三大規則）

### 文字溢出 — 縮小字體

```python
def shrink_font(textbox, new_size_pt):
    for p in textbox.text_frame.paragraphs:
        for r in p.runs:
            r.font.size = Pt(new_size_pt)
```

### 文字溢出 — 擴大容器

```python
def expand_box(shape, delta_h_pt=0, delta_w_pt=0):
    shape.height = shape.height + Pt(delta_h_pt)
    shape.width = shape.width + Pt(delta_w_pt)
```

### 標題間距不足 — 整組下移

```python
def shift_header_group(slide, dy_pt):
    # 將「圓形圖標、頁碼文字、標題、副標、分隔線」統一下移
    for shp in slide.shapes:
        # 頂部色帶（top < 2pt）不動
        if shp.top < Pt(2):
            continue
        # 頁尾（top > 500pt）不動
        if shp.top > Pt(500):
            continue
        # 其餘標題區元素整組下移
        if shp.top < Pt(110):
            shp.top = shp.top + Pt(dy_pt)
```

---

## 12. 完整最小範例

```python
def build_example():
    prs = create_deck()
    total = 3

    # 封面
    cover = prs.slides.add_slide(prs.slide_layouts[6])
    add_cover_bars(cover)
    add_text(cover, 60, 220, 840, 80,
             "2026 年度營運簡報", 48, bold=True,
             color=C_TITLE, align=PP_ALIGN.CENTER)
    add_text(cover, 60, 310, 840, 30,
             "Annual Business Review", 16,
             color=C_SUB, align=PP_ALIGN.CENTER)

    # 內頁 1 — 3 欄卡片
    def body_p2(slide):
        add_three_column_cards(slide, [
            ("使用者成長", "MAU 較去年同期 +32%，亞太區為主要貢獻。"),
            ("營收表現", "Q4 營收 $1.2B，超越預期 8%。"),
            ("產品進展", "新版 API 上線，錯誤率下降 45%。"),
        ])
    add_content_slide(prs, 2, total,
                      "年度重點摘要", "Key Highlights", body_p2)

    # 內頁 2 — 空白模板
    add_content_slide(prs, 3, total,
                      "結語與展望", "Wrap-up & Outlook",
                      lambda s: None)

    prs.save("example.pptx")
    # 掃描與報告
    for a in verify_slides("example.pptx"):
        print(a)
```

---

## 13. 注意事項

- `python-pptx` **無法**真正量測「文字在該字級下的實際像素寬度」；`find_overflow` 的估算僅能抓明顯溢出。**真正的溢出 / 重疊判定仍仰賴 `verify_slide_visual` 的截圖肉眼（或多模態）檢視**。
- `shape.text_frame.auto_size` 預設關閉；開啟後 PowerPoint 會自動縮字，但會違反「字級規範」，**不建議使用**。
- 中文字寬度隨字型而異，Microsoft JhengHei 為比例字型，估算請給 15~20% 緩衝。
- 所有座標 / 尺寸統一用 `Pt()`，避免 EMU 數字不直觀。
