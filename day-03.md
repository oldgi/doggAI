---
title: 📘 Day 3： 來建一個WebUI
---
# 📖 Web UI 開發歷程日誌

*開發日期: 2025/08/02*  
*目標: 從基礎 LLM 環境建立完整的 Web UI 系統*

---

## 🎯 開發目標設定

### 初始需求分析
經過上次的基礎環境建置（RTX 4070 + WSL2 + Ollama），今日目標聚焦於：
1. ✅ 完成 Web UI 界面
2. ✅ 建立文件摘要功能  
3. 🔄 優化使用者體驗

**決策點**: 選擇 Streamlit 作為 Web 框架（易上手、快速開發）

---

## 🚀 第一階段：基礎 Web UI 建立

### v1.0 - 初版實現
```bash
檔案: ~/llm_web_ui.py
功能: 基本聊天界面 + 模型選擇
```

#### ✅ 成功實現：
- 基本聊天對話功能
- 三模型選擇（llama2:7b, llama2:13b, qwen2.5:7b）
- 側邊欄系統狀態

#### ❌ 發現問題：
- **回應時間消失問題**：每次新對話會覆蓋前次的時間資訊
- **相容性錯誤**：`st.sidebar.spinner()` 在舊版 Streamlit 不存在

### 🔧 問題修正路線
**修正 A**: 解決 Streamlit 版本相容性
- 移除 `st.sidebar.spinner()`
- 改用 `st.spinner()` 包裝

**結果**: 界面成功運行，但核心問題仍存在

---

## 🔄 第二階段：用戶體驗優化

### v2.0 - 資訊保留改進
```bash
檔案: ~/llm_web_ui.py (更新)
重點: 解決回應時間消失問題
```

#### 🎯 核心改進：
- **永久時間顯示**：在 `st.session_state.messages` 中儲存 `duration` 資訊
- **模型資訊追蹤**：記錄每個回應使用的模型
- **格式化顯示**：`🤖 模型名稱 | ⏱️ X.XX秒` 格式

#### ✅ 測試結果：
**成功**！回應時間和模型資訊都能永久保留

#### ❌ 新問題浮現：
- **即時顯示缺失**：新對話的即時顯示只有時間，沒有模型資訊
- **布局問題**：右側監控面板會被長對話推到下方

---

## 🎨 第三階段：界面布局優化

### 問題診斷
**核心問題**: 右側系統監控面板不固定，長對話後不可見

### v2.1 - CSS 固定嘗試（失敗分支）
```css
.stColumn:nth-child(2) {
    position: sticky;
    top: 0;
    height: 100vh;
}
```

#### ❌ 結果：
**失敗**！CSS sticky 在 Streamlit 中無效

### v2.2 - 容器高度限制方案（成功分支）
**關鍵洞察**: 改變布局策略，而非強制固定

#### 🔧 解決策略：
1. **調整欄位順序**：右側面板優先處理（`col2` 在前）
2. **限制聊天高度**：`st.container(height=600)` 讓對話區域可滾動
3. **輸入框外置**：保持輸入框始終可見

#### ✅ 測試結果：
**成功**！右側面板固定在頂部，用戶體驗大幅改善

---

## ⚡ 第四階段：效能優化

### 效能問題分析
**發現**: Web UI 回應時間（23.96秒平均）vs 命令列速度測試（0.32-1.85秒）差距巨大

### 診斷過程
#### 速度測試腳本驗證：
```bash
檔案: ~/speed_test.py
結果: 底層 LLM 效能優異
- llama2:7b: 0.36秒
- llama2:13b: 1.85秒  
- qwen2.5:7b: 0.32秒
```

**結論**: 瓶頸在 Streamlit 界面層，非 LLM 本身

### v3.0 - 效能優化版
#### 🚀 優化措施：
1. **Ollama 環境變數優化**：
   - `OLLAMA_KEEP_ALIVE="-1"`（模型永駐記憶體）
   - `OLLAMA_NUM_PARALLEL=1`
   - `OLLAMA_FLASH_ATTENTION=1`

2. **UI 渲染優化**：
   - 減少不必要的 `st.rerun()`
   - 簡化狀態更新邏輯
   - 移除複雜的監控循環

#### ✅ 效能提升：
**llama2:13b**: 48.25秒 → 27.84秒（**提升 42%**）

---

## 📄 第五階段：功能模組整合

### 文件摘要功能開發

#### v4.0 - 獨立摘要工具
```bash
檔案: ~/document_summarizer_v2.py
特色: 命令列 + 互動模式
```

#### v5.0 - Web UI 完整整合
```bash
檔案: ~/llm_complete_ui.py
架構: 多頁面應用
```

**整合挑戰與解決**:
1. **函數定義順序問題**：`NameError: chat_interface not defined`
   - **解決**: 調整函數定義位置，確保在調用前定義

2. **多模組狀態管理**：不同功能間的數據隔離
   - **解決**: 使用不同的 session_state 鍵值

3. **功能切換流暢性**：確保模組間無縫切換
   - **解決**: 統一的側邊欄控制邏輯

---

## 🛤️ 開發路線總結

### 主要發展路線：
```
v1.0 基礎聊天 → v2.0 資訊保留 → v2.2 布局優化 → v3.0 效能優化 → v5.0 功能整合
```

### 關鍵分支決策：
1. **CSS vs 容器方案**：選擇容器高度限制（務實路線）
2. **單頁 vs 多頁應用**：選擇多頁架構（可擴展性）
3. **即時監控 vs 手動更新**：暫時選擇手動（穩定性優先）

---

## 💡 技術突破點

### 成功的解決方案：
1. **Session State 數據持久化**：完美解決資訊保留問題
2. **雙欄布局 + 容器限高**：創新的固定面板解決方案  
3. **環境變數優化**：大幅提升 LLM 回應速度
4. **模組化架構設計**：為後續功能擴展奠定基礎

### 失敗但有價值的嘗試：
1. **CSS Sticky 定位**：學習到 Streamlit 的 CSS 限制
2. **複雜即時監控**：了解到 Streamlit 背景任務限制

---

## 📊 最終成果

### 功能清單：
- ✅ **智能聊天**：三模型支援，完整資訊追蹤
- ✅ **文件摘要**：命令列 + Web UI 雙界面
- ✅ **使用統計**：詳細的使用數據分析
- ✅ **效能監控**：手動 GPU 狀態檢查

### 技術指標：
- **回應速度**：27.84秒（優化後）
- **界面穩定性**：無閃退，流暢切換
- **功能完整性**：核心需求 100% 實現

---

## 🔮 開發心得

### 成功因素：
1. **漸進式開發**：每個版本解決一個核心問題
2. **用戶導向思維**：優先解決體驗痛點
3. **實測驗證**：每次改進都經過實際測試
4. **備選方案準備**：CSS 失敗後快速轉向容器方案

### 技術債務：
1. **GPU 即時監控**：仍需後續專門開發
2. **更精細的效能調優**：還有優化空間
3. **移動端適配**：未來可考慮

---

*開發總時長: ~6 小時*  
*主要版本迭代: 5 次*  
*核心問題解決: 4 個*  
*最終狀態: 生產就緒* ✅

**關鍵成功路線**: 基礎實現 → 體驗優化 → 效能提升 → 功能整合

## 附錄
```
cat > ~/llm_complete_ui.py << 'EOF'
import streamlit as st
import ollama
import time
import os
import tempfile
from datetime import datetime

st.set_page_config(page_title="LLM 完整助手", layout="wide")
st.title("🤖 我的完整 LLM 助手")

# 初始化 session state
if "chat_messages" not in st.session_state:
    st.session_state.chat_messages = []
if "summary_history" not in st.session_state:
    st.session_state.summary_history = []

def generate_summary(content, model, detail_level):
    """生成摘要"""
    # 根據詳細程度設定提示詞
    prompts = {
        "detailed": f"""
請詳細摘要以下內容，要求：
1. **主要內容概述**：用1-2句話概括整體內容
2. **核心要點**：提取5-8個關鍵要點，每個要點包含具體細節
3. **重要數據/資訊**：列出所有重要的數字、時間、名稱等
4. **結論或展望**：總結主要結論
5. **用中文回應**，格式清晰

內容：
{content}
""",
        "brief": f"""
請簡要摘要以下內容：
1. 用3-4個要點概括主要內容
2. 保持簡潔，每個要點1-2句話
3. 用中文回應

內容：
{content}
""",
        "outline": f"""
請為以下內容建立大綱：
1. 按照層次結構組織內容
2. 使用標題和子標題
3. 突出重點章節
4. 用中文回應

內容：
{content}
"""
    }
    
    start_time = time.time()
    try:
        response = ollama.chat(model=model, messages=[
            {"role": "user", "content": prompts[detail_level]}
        ])
        duration = time.time() - start_time
        
        return {
            "success": True,
            "summary": response['message']['content'],
            "duration": duration
        }
    except Exception as e:
        duration = time.time() - start_time
        return {
            "success": False,
            "error": str(e),
            "duration": duration
        }

def chat_interface(model):
    """聊天界面"""
    col1, col2 = st.columns([2, 1])
    
    with col2:
        st.subheader("📊 聊天狀態")
        
        # 統計資訊
        user_messages = [m for m in st.session_state.chat_messages if m['role'] == 'user']
        assistant_messages = [m for m in st.session_state.chat_messages if m['role'] == 'assistant']
        
        st.metric("💬 對話輪數", len(user_messages))
        
        if assistant_messages:
            avg_time = sum(m.get('duration', 0) for m in assistant_messages) / len(assistant_messages)
            st.metric("⏱️ 平均回應時間", f"{avg_time:.2f}秒")
            
            times = [m.get('duration', 0) for m in assistant_messages]
            st.metric("🏃 最快回應", f"{min(times):.2f}秒")
        
        st.markdown("---")
        
        if st.button("🗑️ 清除聊天記錄"):
            st.session_state.chat_messages = []
            st.rerun()
    
    with col1:
        st.subheader(f"💬 聊天區域 - {model}")
        
        # 聊天容器
        chat_container = st.container(height=500)
        
        with chat_container:
            for message in st.session_state.chat_messages:
                with st.chat_message(message["role"]):
                    st.markdown(message["content"])
                    if message["role"] == "assistant" and "duration" in message:
                        info_parts = [f"🤖 {message.get('model', model)}"]
                        if "duration" in message:
                            color = "🟢" if message['duration'] < 5 else "🟡" if message['duration'] < 15 else "🔴"
                            info_parts.append(f"⏱️ {message['duration']:.2f}秒 {color}")
                        st.caption(" | ".join(info_parts))
        
        # 聊天輸入
        if prompt := st.chat_input("請輸入你的問題..."):
            st.session_state.chat_messages.append({"role": "user", "content": prompt})
            
            start_time = time.time()
            response = ollama.chat(model=model, messages=[
                {"role": "user", "content": prompt}
            ])
            duration = time.time() - start_time
            
            st.session_state.chat_messages.append({
                "role": "assistant", 
                "content": response['message']['content'],
                "model": model,
                "duration": duration
            })
            
            st.rerun()

def document_summary_interface(model):
    """文件摘要界面"""
    st.subheader("📄 文件摘要工具")
    
    # 摘要設定
    col1, col2 = st.columns([2, 1])
    
    with col2:
        st.subheader("⚙️ 摘要設定")
        
        detail_level = st.selectbox(
            "摘要詳細程度",
            ["detailed", "brief", "outline"],
            format_func=lambda x: {
                "detailed": "📋 詳細摘要", 
                "brief": "📝 簡要摘要", 
                "outline": "📑 大綱模式"
            }[x]
        )
        
        st.markdown("---")
        st.subheader("📊 摘要歷史")
        
        if st.session_state.summary_history:
            for i, history in enumerate(reversed(st.session_state.summary_history[-5:])):
                with st.expander(f"📄 {history['filename']} ({history['timestamp']})"):
                    st.write(f"**模型**: {history['model']}")
                    st.write(f"**類型**: {history['detail_level']}")
                    st.write(f"**時間**: {history['duration']:.2f}秒")
                    st.text_area("摘要內容", history['summary'], height=200, key=f"summary_view_{i}")
        else:
            st.info("還沒有摘要記錄")
        
        if st.button("🗑️ 清除摘要歷史"):
            st.session_state.summary_history = []
            st.rerun()
    
    with col1:
        st.subheader("📁 文件上傳與摘要")
        
        # 文件上傳
        uploaded_file = st.file_uploader(
            "上傳文件", 
            type=['txt', 'md', 'py', 'js', 'html', 'css', 'json'],
            help="支援文字格式文件"
        )
        
        # 或者文字直接輸入
        st.markdown("**或者直接輸入文字：**")
        text_input = st.text_area("請貼上要摘要的文字內容", height=200)
        
        # 摘要按鈕
        if st.button("🚀 開始摘要", type="primary"):
            content = ""
            filename = ""
            
            if uploaded_file is not None:
                # 處理上傳文件
                try:
                    content = str(uploaded_file.read(), "utf-8")
                    filename = uploaded_file.name
                except:
                    st.error("❌ 無法讀取文件，請檢查文件格式")
                    st.stop()
            elif text_input.strip():
                # 處理直接輸入
                content = text_input.strip()
                filename = "直接輸入文字"
            else:
                st.warning("⚠️ 請上傳文件或輸入文字")
                st.stop()
            
            if content:
                # 執行摘要
                with st.spinner(f"🤖 {model} 正在分析文件..."):
                    summary_result = generate_summary(content, model, detail_level)
                
                if summary_result["success"]:
                    # 顯示結果
                    st.success(f"✅ 摘要完成！耗時 {summary_result['duration']:.2f} 秒")
                    
                    # 顯示摘要
                    with st.container():
                        st.markdown("### 📋 摘要結果")
                        st.markdown(summary_result["summary"])
                    
                    # 保存到歷史
                    st.session_state.summary_history.append({
                        "filename": filename,
                        "model": model,
                        "detail_level": detail_level,
                        "duration": summary_result["duration"],
                        "summary": summary_result["summary"],
                        "timestamp": datetime.now().strftime("%H:%M:%S")
                    })
                    
                    # 下載按鈕
                    summary_text = f"""📚 文件摘要報告
{"=" * 50}
原始文件: {filename}
摘要時間: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
使用模型: {model}
摘要類型: {detail_level}
處理時間: {summary_result['duration']:.2f} 秒
{"=" * 50}

{summary_result["summary"]}"""
                    
                    st.download_button(
                        label="💾 下載摘要",
                        data=summary_text,
                        file_name=f"{filename}_summary_{datetime.now().strftime('%Y%m%d_%H%M%S')}.txt",
                        mime="text/plain"
                    )
                else:
                    st.error(f"❌ 摘要失敗: {summary_result['error']}")

def statistics_interface():
    """統計界面"""
    st.subheader("📊 使用統計")
    
    col1, col2 = st.columns(2)
    
    with col1:
        st.markdown("### 💬 聊天統計")
        
        if st.session_state.chat_messages:
            user_msgs = [m for m in st.session_state.chat_messages if m['role'] == 'user']
            assistant_msgs = [m for m in st.session_state.chat_messages if m['role'] == 'assistant']
            
            st.metric("總對話輪數", len(user_msgs))
            
            if assistant_msgs:
                avg_time = sum(m.get('duration', 0) for m in assistant_msgs) / len(assistant_msgs)
                st.metric("平均回應時間", f"{avg_time:.2f}秒")
                
                # 模型使用統計
                model_usage = {}
                for msg in assistant_msgs:
                    model = msg.get('model', 'unknown')
                    model_usage[model] = model_usage.get(model, 0) + 1
                
                st.markdown("**模型使用次數:**")
                for model, count in model_usage.items():
                    st.write(f"- {model}: {count} 次")
        else:
            st.info("還沒有聊天記錄")
    
    with col2:
        st.markdown("### 📄 摘要統計")
        
        if st.session_state.summary_history:
            st.metric("總摘要數量", len(st.session_state.summary_history))
            
            # 摘要類型統計
            type_usage = {}
            total_time = 0
            
            for summary in st.session_state.summary_history:
                detail_type = summary['detail_level']
                type_usage[detail_type] = type_usage.get(detail_type, 0) + 1
                total_time += summary.get('duration', 0)
            
            if st.session_state.summary_history:
                avg_summary_time = total_time / len(st.session_state.summary_history)
                st.metric("平均摘要時間", f"{avg_summary_time:.2f}秒")
            
            st.markdown("**摘要類型分布:**")
            for summary_type, count in type_usage.items():
                st.write(f"- {summary_type}: {count} 次")
        else:
            st.info("還沒有摘要記錄")

# 側邊欄 - 功能選擇
st.sidebar.title("🎯 功能選單")
app_mode = st.sidebar.selectbox(
    "選擇功能", 
    ["💬 智能聊天", "📄 文件摘要", "📊 使用統計"]
)

# 模型選擇（全局）
models = ["qwen2.5:7b", "llama2:7b", "llama2:13b"]
selected_model = st.sidebar.selectbox("🤖 選擇模型", models)

# 主界面根據選擇的功能顯示不同內容
if app_mode == "💬 智能聊天":
    chat_interface(selected_model)
elif app_mode == "📄 文件摘要":
    document_summary_interface(selected_model)
elif app_mode == "📊 使用統計":
    statistics_interface()

# 側邊欄底部資訊
st.sidebar.markdown("---")
st.sidebar.markdown("🚀 **RTX 4070 + WSL2**")
st.sidebar.markdown(f"🎯 **當前模型**: {selected_model}")
st.sidebar.markdown(f"⏰ **當前時間**: {datetime.now().strftime('%H:%M:%S')}")
EOF
```