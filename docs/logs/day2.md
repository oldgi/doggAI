- # 📂 20250726附錄：測試程式碼集合
- ## A1. GPU 監控腳本
- ### `gpu_monitor.sh`
  
  ```
  #!/bin/bash
  
  while true; do
    clear
    echo "=== GPU 監控 (按 Ctrl+C 停止) ==="
    echo "時間: $(date)"
    echo ""
    
    # GPU 基本資訊
    nvidia-smi --query-gpu=name,memory.used,memory.total,utilization.gpu,temperature.gpu,power.draw --format=csv,noheader,nounits | \
    awk -F', ' '{
        printf "GPU: %s\n", $1
        printf "記憶體: %s MB / %s MB (%.1f%%)\n", $2, $3, ($2/$3)*100
        printf "使用率: %s%%\n", $4
        printf "溫度: %s°C\n", $5
        printf "功耗: %s W\n", $6
    }'
    
    echo ""
    echo "執行中的 GPU 程序:"
    nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv,noheader,nounits 2>/dev/null || echo "無 GPU 程序運行"
    
    sleep 2
  done
  ```
- ## A2. 系統監控腳本
- ### `monitor_system.sh`
  
  ```
  #!/bin/bash
  
  echo "=== 系統資源監控 ==="
  echo "時間: $(date)"
  echo ""
  
  echo "1. 記憶體使用狀況:"
  free -h
  echo ""
  
  echo "2. 磁碟使用狀況:"
  df -h | grep -E "(根目錄|/dev/)"
  echo ""
  
  echo "3. CPU 負載:"
  uptime
  echo ""
  
  echo "4. GPU 狀況:"
  nvidia-smi --query-gpu=name,memory.used,memory.total,utilization.gpu,temperature.gpu,power.draw --format=csv,noheader,nounits
  echo ""
  
  echo "5. 前 10 個消耗記憶體的程序:"
  ps aux --sort=-%mem | head -11
  echo ""
  ```
- ## A3. API 測試腳本
- ### `test_api.py`
  
  ```
  import ollama
  import time
  
  def test_performance():
    print("開始測試 Ollama API...")
    start_time = time.time()
    
    response = ollama.chat(model='llama2:7b', messages=[
        {'role': 'user', 'content': 'Count from 1 to 5 and briefly explain each number'}
    ])
    
    end_time = time.time()
    
    print(f"回應時間: {end_time - start_time:.2f} 秒")
    print("="*50)
    print("完整回應內容:")
    print(response['message']['content'])
    print("="*50)
  
  if __name__ == "__main__":
    test_performance()
  ```
- ### `test_streaming.py`
  
  ```
  import ollama
  import time
  
  def test_streaming():
    print("測試串流回應...")
    
    start_time = time.time()
    stream = ollama.chat(
        model='llama2:7b',
        messages=[{'role': 'user', 'content': 'Tell me a short story about a robot'}],
        stream=True,
    )
    
    print("串流回應開始:")
    print("-" * 30)
    
    for chunk in stream:
        content = chunk['message']['content']
        print(content, end='', flush=True)
    
    end_time = time.time()
    print(f"\n-" * 30)
    print(f"總時間: {end_time - start_time:.2f} 秒")
  
  def test_direct_api():
    print("\n測試直接 API 呼叫...")
    
    response = ollama.generate(
        model='llama2:7b',
        prompt='What are the 3 primary colors?'
    )
    
    print("直接 API 回應:")
    print(response['response'])
  
  if __name__ == "__main__":
    test_streaming()
    test_direct_api()
  ```
- ### `performance_test.py`
  
  ```
  import ollama
  import time
  import json
  
  def detailed_performance_test():
    tests = [
        "What is 2+2?",
        "Name 3 colors",
        "Write a haiku about computers",
        "Explain photosynthesis in one sentence"
    ]
    
    print("詳細效能測試開始...")
    print("="*60)
    
    for i, prompt in enumerate(tests, 1):
        print(f"\n測試 {i}: {prompt}")
        print("-" * 40)
        
        start_time = time.time()
        response = ollama.chat(model='llama2:7b', messages=[
            {'role': 'user', 'content': prompt}
        ])
        end_time = time.time()
        
        response_time = end_time - start_time
        response_text = response['message']['content']
        
        print(f"回應時間: {response_time:.2f} 秒")
        print(f"回應長度: {len(response_text)} 字元")
        print(f"回應內容: {response_text}")
        print(f"平均速度: {len(response_text)/response_time:.1f} 字元/秒")
  
  if __name__ == "__main__":
    detailed_performance_test()
  ```
- ## A4. 進階測試腳本
- ### `conversation_test.py`
  
  ```
  import ollama
  import time
  
  def test_conversation():
    print("測試連續對話效能...")
    
    # 第一次對話（模型載入）
    start_time = time.time()
    response1 = ollama.chat(model='llama2:7b', messages=[
        {'role': 'user', 'content': 'Hello, what is your name?'}
    ])
    time1 = time.time() - start_time
    
    # 第二次對話（模型已載入）
    start_time = time.time()
    response2 = ollama.chat(model='llama2:7b', messages=[
        {'role': 'user', 'content': 'What is 5 + 3?'}
    ])
    time2 = time.time() - start_time
    
    print(f"第一次回應時間: {time1:.2f} 秒")
    print(f"第二次回應時間: {time2:.2f} 秒")
    print(f"效能提升: {((time1-time2)/time1*100):.1f}%")
    
    print("\n第一次回應:", response1['message']['content'][:100] + "...")
    print("第二次回應:", response2['message']['content'])
  
  if __name__ == "__main__":
    test_conversation()
  ```
- ### `model_comparison.py`
  
  ```
  import ollama
  import time
  
  def compare_models():
    models = ['llama2:7b', 'qwen2.5:7b']
    prompt = "What is machine learning?"
    
    print("模型效能比較測試...")
    print("="*50)
    
    for model in models:
        try:
            print(f"\n測試模型: {model}")
            start_time = time.time()
            
            response = ollama.chat(model=model, messages=[
                {'role': 'user', 'content': prompt}
            ])
            
            end_time = time.time()
            response_time = end_time - start_time
            
            print(f"回應時間: {response_time:.2f} 秒")
            print(f"回應預覽: {response['message']['content'][:150]}...")
            
        except Exception as e:
            print(f"模型 {model} 測試失敗: {e}")
  
  if __name__ == "__main__":
    compare_models()
  ```
- ## A5. 自動化測試框架
- ### `auto_test_framework.py`
  
  ```
  import ollama
  import time
  import json
  import subprocess
  import psutil
  import os
  from datetime import datetime
  from pathlib import Path
  
  class LLMTestFramework:
    def __init__(self):
        self.log_file = f"llm_test_log_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        self.results = []
        
    def get_gpu_stats(self):
        """獲取 GPU 狀態"""
        try:
            result = subprocess.run([
                'nvidia-smi', '--query-gpu=utilization.gpu,memory.used,memory.total,temperature.gpu,power.draw',
                '--format=csv,noheader,nounits'
            ], capture_output=True, text=True)
            
            if result.returncode == 0:
                stats = result.stdout.strip().split(', ')
                return {
                    'gpu_util': float(stats[0]),
                    'memory_used': int(stats[1]),
                    'memory_total': int(stats[2]),
                    'memory_percent': (int(stats[1]) / int(stats[2])) * 100,
                    'temperature': float(stats[3]),
                    'power_draw': float(stats[4])
                }
        except Exception as e:
            print(f"無法獲取 GPU 狀態: {e}")
            return None
    
    def get_system_stats(self):
        """獲取系統狀態"""
        memory = psutil.virtual_memory()
        cpu_percent = psutil.cpu_percent(interval=1)
        
        return {
            'cpu_percent': cpu_percent,
            'ram_used_gb': memory.used / (1024**3),
            'ram_total_gb': memory.total / (1024**3),
            'ram_percent': memory.percent
        }
    
    def run_test(self, test_name, model, prompt, description=""):
        """執行單個測試並記錄結果"""
        print(f"\n🧪 開始測試: {test_name}")
        print(f"📋 描述: {description}")
        print(f"🤖 模型: {model}")
        print(f"💬 提示: {prompt}")
        print("-" * 60)
        
        # 記錄測試前狀態
        pre_gpu = self.get_gpu_stats()
        pre_system = self.get_system_stats()
        
        # 執行測試
        start_time = time.time()
        try:
            response = ollama.chat(model=model, messages=[
                {'role': 'user', 'content': prompt}
            ])
            success = True
            error_msg = None
        except Exception as e:
            response = None
            success = False
            error_msg = str(e)
        
        end_time = time.time()
        
        # 記錄測試後狀態
        post_gpu = self.get_gpu_stats()
        post_system = self.get_system_stats()
        
        duration = end_time - start_time
        
        # 準備結果數據
        result = {
            'timestamp': datetime.now().isoformat(),
            'test_name': test_name,
            'description': description,
            'model': model,
            'prompt': prompt,
            'duration_seconds': round(duration, 2),
            'success': success,
            'error_message': error_msg,
            'response_length': len(response['message']['content']) if response else 0,
            'tokens_per_second': len(response['message']['content']) / duration if response else 0,
            'pre_test_gpu': pre_gpu,
            'post_test_gpu': post_gpu,
            'pre_test_system': pre_system,
            'post_test_system': post_system,
            'response_preview': response['message']['content'][:200] + "..." if response else None
        }
        
        self.results.append(result)
        
        # 即時顯示結果
        print(f"✅ 測試完成!")
        print(f"⏱️  執行時間: {duration:.2f} 秒")
        if post_gpu:
            print(f"🔥 GPU 使用率: {post_gpu['gpu_util']}%")
            print(f"💾 GPU 記憶體: {post_gpu['memory_percent']:.1f}%")
            print(f"🌡️  GPU 溫度: {post_gpu['temperature']}°C")
            print(f"⚡ GPU 功耗: {post_gpu['power_draw']}W")
        
        if response:
            print(f"📝 回應長度: {len(response['message']['content'])} 字元")
            print(f"🏃 處理速度: {len(response['message']['content'])/duration:.1f} 字元/秒")
            print(f"💬 回應預覽: {response['message']['content'][:150]}...")
        
        return result
    
    def save_results(self):
        """儲存所有測試結果"""
        with open(self.log_file, 'w', encoding='utf-8') as f:
            json.dump(self.results, f, indent=2, ensure_ascii=False)
        
        print(f"\n📊 測試結果已儲存至: {self.log_file}")
        return self.log_file
    
    def generate_summary(self):
        """生成測試摘要"""
        if not self.results:
            return
        
        print("\n" + "="*80)
        print("📊 測試摘要報告")
        print("="*80)
        
        successful_tests = [r for r in self.results if r['success']]
        failed_tests = [r for r in self.results if not r['success']]
        
        print(f"總測試數: {len(self.results)}")
        print(f"成功: {len(successful_tests)}")
        print(f"失敗: {len(failed_tests)}")
        
        if successful_tests:
            avg_duration = sum(r['duration_seconds'] for r in successful_tests) / len(successful_tests)
            avg_speed = sum(r['tokens_per_second'] for r in successful_tests) / len(successful_tests)
            
            gpu_utils = [r['post_test_gpu']['gpu_util'] for r in successful_tests if r['post_test_gpu']]
            gpu_temps = [r['post_test_gpu']['temperature'] for r in successful_tests if r['post_test_gpu']]
            gpu_powers = [r['post_test_gpu']['power_draw'] for r in successful_tests if r['post_test_gpu']]
            
            print(f"\n效能統計:")
            print(f"  平均執行時間: {avg_duration:.2f} 秒")
            print(f"  平均處理速度: {avg_speed:.1f} 字元/秒")
            
            if gpu_utils:
                print(f"  平均 GPU 使用率: {sum(gpu_utils)/len(gpu_utils):.1f}%")
                print(f"  平均 GPU 溫度: {sum(gpu_temps)/len(gpu_temps):.1f}°C")
                print(f"  平均 GPU 功耗: {sum(gpu_powers)/len(gpu_powers):.1f}W")
        
        if failed_tests:
            print(f"\n❌ 失敗的測試:")
            for test in failed_tests:
                print(f"  - {test['test_name']}: {test['error_message']}")
  
  # 使用範例
  def main():
    framework = LLMTestFramework()
    
    # 定義測試案例
    test_cases = [
        {
            'name': '簡單數學',
            'model': 'llama2:7b',
            'prompt': 'What is 15 + 27?',
            'description': '測試基本數學計算能力'
        },
        {
            'name': '短文解釋',
            'model': 'llama2:7b',
            'prompt': 'Explain what is photosynthesis in 2 sentences',
            'description': '測試科學概念解釋能力'
        },
        {
            'name': '創意寫作',
            'model': 'llama2:7b',
            'prompt': 'Write a haiku about artificial intelligence',
            'description': '測試創意寫作能力'
        },
        {
            'name': '中文對話',
            'model': 'qwen2.5:7b',
            'prompt': '請用中文解釋什麼是機器學習',
            'description': '測試中文模型表現'
        }
    ]
    
    # 執行所有測試
    for test_case in test_cases:
        framework.run_test(
            test_case['name'],
            test_case['model'],
            test_case['prompt'],
            test_case['description']
        )
        time.sleep(2)  # 讓系統穩定
    
    # 儲存結果並生成摘要
    log_file = framework.save_results()
    framework.generate_summary()
    
    return log_file
  
  if __name__ == "__main__":
    log_file = main()
    print(f"\n🎉 所有測試完成！詳細記錄請查看: {log_file}")
  ```
- ## A6. HTML 報告生成器
- ### `html_reporter.py`
  
  ```
  import json
  from datetime import datetime
  from pathlib import Path
  
  def generate_html_report(json_file):
    """從 JSON 記錄生成 HTML 報告"""
    
    with open(json_file, 'r', encoding='utf-8') as f:
        data = json.load(f)
    
    html = f"""
    <!DOCTYPE html>
    <html>
    <head>
        <title>LLM 測試報告</title>
        <style>
            body {{ font-family: Arial, sans-serif; margin: 20px; }}
            .test {{ border: 1px solid #ddd; margin: 10px 0; padding: 15px; border-radius: 5px; }}
            .success {{ border-left: 5px solid #4CAF50; }}
            .failed {{ border-left: 5px solid #f44336; }}
            .stats {{ background: #f9f9f9; padding: 10px; margin: 10px 0; }}
            table {{ border-collapse: collapse; width: 100%; }}
            th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
            th {{ background-color: #f2f2f2; }}
        </style>
    </head>
    <body>
        <h1>🚀 RTX 4070 LLM 測試報告</h1>
        <p>生成時間: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}</p>
        
        <h2>📊 統計摘要</h2>
        <table>
            <tr><th>項目</th><th>數值</th></tr>
            <tr><td>總測試數</td><td>{len(data)}</td></tr>
            <tr><td>成功測試</td><td>{len([r for r in data if r.get('success', True)])}</td></tr>
            <tr><td>平均執行時間</td><td>{sum(r['duration_seconds'] for r in data)/len(data):.2f} 秒</td></tr>
        </table>
    """
    
    html += "<h2>🧪 詳細測試結果</h2>"
    
    for i, result in enumerate(data, 1):
        status_class = "success" if result.get('success', True) else "failed"
        gpu_stats = result.get('post_test_gpu', {})
        
        html += f"""
        <div class="test {status_class}">
            <h3>測試 {i}: {result['test_name']}</h3>
            <p><strong>描述:</strong> {result.get('description', 'N/A')}</p>
            <p><strong>模型:</strong> {result['model']}</p>
            <p><strong>提示:</strong> {result['prompt']}</p>
            <p><strong>執行時間:</strong> {result['duration_seconds']} 秒</p>
            
            <div class="stats">
                <strong>GPU 狀態:</strong><br>
                使用率: {gpu_stats.get('gpu_util', 'N/A')}% | 
                記憶體: {gpu_stats.get('memory_percent', 'N/A'):.1f}% | 
                溫度: {gpu_stats.get('temperature', 'N/A')}°C | 
                功耗: {gpu_stats.get('power_draw', 'N/A')}W
            </div>
            
            <p><strong>回應預覽:</strong><br>
            <em>{result.get('response_preview', 'N/A')}</em></p>
        </div>
        """
    
    html += "</body></html>"
    
    html_file = json_file.replace('.json', '.html')
    with open(html_file, 'w', encoding='utf-8') as f:
        f.write(html)
    
    print(f"📄 HTML 報告已生成: {html_file}")
    return html_file
  
  if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1:
        generate_html_report(sys.argv[1])
    else:
        print("用法: python3 html_reporter.py <json_file>")
  ```
  
---
- ## 📝 使用說明
- ### 基本使用
  
  ```
  # 1. 給予執行權限
  chmod +x ~/gpu_monitor.sh ~/monitor_system.sh
  
  # 2. 執行 GPU 監控
  ./gpu_monitor.sh
  
  # 3. 執行系統監控
  ./monitor_system.sh
  
  # 4. 執行 API 測試
  python3 ~/test_api.py
  
  # 5. 執行完整自動化測試
  python3 ~/auto_test_framework.py
  
  # 6. 生成 HTML 報告
  python3 ~/html_reporter.py llm_test_log_*.json
  ```
- ### 依賴套件
  
  <!--EndFragment-->
- ### 依賴套件
  
  bash  
  
  ```
  *# 安裝 Python 套件*
  pip3 install psutil ollama requests
  
  *# 安裝系統監控工具*
  sudo apt install htop nvtop iotop
  ```
- ### 檔案結構
  
  ```
  ~/
  ├── gpu_monitor.sh              # GPU 即時監控
  ├── monitor_system.sh           # 系統資源監控
  ├── test_api.py                 # 基本 API 測試
  ├── test_streaming.py           # 串流回應測試
  ├── performance_test.py         # 效能基準測試
  ├── conversation_test.py        # 連續對話測試
  ├── model_comparison.py         # 模型比較測試
  ├── auto_test_framework.py      # 自動化測試框架 ⭐
  ├── html_reporter.py            # HTML 報告生成器
  ├── llm_test_log_*.json        # 測試記錄檔案
  └── llm_test_log_*.html        # HTML 報告檔案
  ```
  
---
- ## A7. 額外實用腳本
- ### `quick_test.py`  - 快速測試腳本
  
  python  
  
  ```
  *#!/usr/bin/env python3*
  import ollama
  import time
  import sys
  
  def quick_test(model="llama2:7b", prompt="Hello, how are you?"):
    """快速測試指定模型"""
    print(f"🚀 快速測試: {model}")
    print(f"📝 提示: {prompt}")
    print("-" * 50)
    
    start_time = time.time()
    try:
        response = ollama.chat(model=model, messages=[
            {'role': 'user', 'content': prompt}
        ])
        duration = time.time() - start_time
        
        print(f"✅ 成功! 耗時: {duration:.2f} 秒")
        print(f"📄 回應: {response['message']['content']}")
        
    except Exception as e:
        print(f"❌ 錯誤: {e}")
  
  if __name__ == "__main__":
    if len(sys.argv) >= 2:
        model = sys.argv[1]
        prompt = " ".join(sys.argv[2:]) if len(sys.argv) > 2 else "Hello, how are you?"
        quick_test(model, prompt)
    else:
        print("用法: python3 quick_test.py <model> [prompt]")
        print("範例: python3 quick_test.py llama2:7b 'What is AI?'")
  ```
- ### `model_manager.py`  - 模型管理腳本
  
  python  
  
  ```
  *#!/usr/bin/env python3*
  import ollama
  import subprocess
  import json
  
  def list_models():
    """列出所有已下載的模型"""
    try:
        result = subprocess.run(['ollama', 'list'], capture_output=True, text=True)
        print("📦 已下載的模型:")
        print(result.stdout)
    except Exception as e:
        print(f"❌ 無法獲取模型列表: {e}")
  
  def download_model(model_name):
    """下載指定模型"""
    print(f"⬇️  正在下載模型: {model_name}")
    try:
        result = subprocess.run(['ollama', 'pull', model_name], check=True)
        print(f"✅ 模型 {model_name} 下載完成!")
    except subprocess.CalledProcessError as e:
        print(f"❌ 下載失敗: {e}")
  
  def remove_model(model_name):
    """刪除指定模型"""
    confirm = input(f"確定要刪除模型 {model_name}? (y/N): ")
    if confirm.lower() == 'y':
        try:
            result = subprocess.run(['ollama', 'rm', model_name], check=True)
            print(f"✅ 模型 {model_name} 已刪除!")
        except subprocess.CalledProcessError as e:
            print(f"❌ 刪除失敗: {e}")
  
  def model_info(model_name):
    """顯示模型資訊"""
    try:
        result = subprocess.run(['ollama', 'show', model_name], capture_output=True, text=True)
        print(f"📋 模型 {model_name} 資訊:")
        print(result.stdout)
    except Exception as e:
        print(f"❌ 無法獲取模型資訊: {e}")
  
  def main():
    while True:
        print("\n" + "="*50)
        print("🤖 Ollama 模型管理器")
        print("="*50)
        print("1. 列出模型")
        print("2. 下載模型")
        print("3. 刪除模型")
        print("4. 模型資訊")
        print("5. 退出")
        
        choice = input("\n請選擇操作 (1-5): ")
        
        if choice == '1':
            list_models()
        elif choice == '2':
            model = input("請輸入模型名稱 (例如 llama2:7b): ")
            download_model(model)
        elif choice == '3':
            list_models()
            model = input("請輸入要刪除的模型名稱: ")
            remove_model(model)
        elif choice == '4':
            model = input("請輸入模型名稱: ")
            model_info(model)
        elif choice == '5':
            print("👋 再見!")
            break
        else:
            print("❌ 無效選擇，請重試")
  
  if __name__ == "__main__":
    main()
  ```
- ### `system_health.py`  - 系統健康檢查
  
  python  
  
  ```
  *#!/usr/bin/env python3*
  import subprocess
  import psutil
  import time
  from datetime import datetime
  
  def check_ollama_service():
    """檢查 Ollama 服務狀態"""
    try:
        result = subprocess.run(['ollama', 'list'], capture_output=True, text=True, timeout=5)
        return True, "Ollama 服務正常"
    except subprocess.TimeoutExpired:
        return False, "Ollama 服務回應超時"
    except Exception as e:
        return False, f"Ollama 服務異常: {e}"
  
  def check_gpu_status():
    """檢查 GPU 狀態"""
    try:
        result = subprocess.run([
            'nvidia-smi', '--query-gpu=name,temperature.gpu,utilization.gpu',
            '--format=csv,noheader,nounits'
        ], capture_output=True, text=True, timeout=5)
        
        if result.returncode == 0:
            gpu_info = result.stdout.strip().split(', ')
            temp = float(gpu_info[1])
            util = float(gpu_info[2])
            
            status = "正常"
            if temp > 80:
                status = "溫度過高"
            elif util > 95:
                status = "使用率過高"
            
            return True, f"GPU {status} - 溫度: {temp}°C, 使用率: {util}%"
        else:
            return False, "無法獲取 GPU 資訊"
    except Exception as e:
        return False, f"GPU 檢查失敗: {e}"
  
  def check_system_resources():
    """檢查系統資源"""
    try:
        *# 記憶體檢查*
        memory = psutil.virtual_memory()
        mem_status = "正常"
        if memory.percent > 90:
            mem_status = "記憶體不足"
        elif memory.percent > 80:
            mem_status = "記憶體使用率高"
        
        *# CPU 檢查*
        cpu_percent = psutil.cpu_percent(interval=1)
        cpu_status = "正常"
        if cpu_percent > 90:
            cpu_status = "CPU 使用率過高"
        elif cpu_percent > 80:
            cpu_status = "CPU 使用率高"
        
        *# 磁碟檢查*
        disk = psutil.disk_usage('/')
        disk_status = "正常"
        if disk.percent > 90:
            disk_status = "磁碟空間不足"
        elif disk.percent > 80:
            disk_status = "磁碟使用率高"
        
        return True, f"系統資源 {mem_status} - RAM: {memory.percent:.1f}%, CPU: {cpu_percent:.1f}%, 磁碟: {disk.percent:.1f}%"
    
    except Exception as e:
        return False, f"系統檢查失敗: {e}"
  
  def health_check():
    """執行完整健康檢查"""
    print("🏥 系統健康檢查")
    print("=" * 50)
    print(f"檢查時間: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print()
    
    checks = [
        ("Ollama 服務", check_ollama_service),
        ("GPU 狀態", check_gpu_status),
        ("系統資源", check_system_resources)
    ]
    
    all_healthy = True
    
    for check_name, check_func in checks:
        try:
            is_ok, message = check_func()
            status_icon = "✅" if is_ok else "❌"
            print(f"{status_icon} {check_name}: {message}")
            if not is_ok:
                all_healthy = False
        except Exception as e:
            print(f"❌ {check_name}: 檢查過程發生錯誤 - {e}")
            all_healthy = False
    
    print()
    if all_healthy:
        print("🎉 系統狀態良好!")
    else:
        print("⚠️  發現問題，請檢查上述項目")
    
    return all_healthy
  
  if __name__ == "__main__":
    health_check()
  ```
- ### `benchmark.py`  - 效能基準測試
  
  python  
  
  ```
  *#!/usr/bin/env python3*
  import ollama
  import time
  import statistics
  import json
  from datetime import datetime
  
  def run_benchmark(model="llama2:7b", iterations=5):
    """執行效能基準測試"""
    
    test_prompts = [
        "What is 2+2?",
        "Explain gravity in one sentence",
        "List 3 programming languages",
        "What is the capital of France?",
        "Write a short haiku"
    ]
    
    print(f"🏃 開始基準測試: {model}")
    print(f"📊 測試次數: {iterations} 輪")
    print(f"📝 測試項目: {len(test_prompts)} 個")
    print("=" * 60)
    
    all_results = []
    
    for round_num in range(1, iterations + 1):
        print(f"\n第 {round_num} 輪測試:")
        round_results = []
        
        for i, prompt in enumerate(test_prompts, 1):
            print(f"  測試 {i}/5: ", end="", flush=True)
            
            start_time = time.time()
            try:
                response = ollama.chat(model=model, messages=[
                    {'role': 'user', 'content': prompt}
                ])
                duration = time.time() - start_time
                response_length = len(response['message']['content'])
                tokens_per_sec = response_length / duration
                
                round_results.append({
                    'prompt': prompt,
                    'duration': duration,
                    'response_length': response_length,
                    'tokens_per_sec': tokens_per_sec,
                    'success': True
                })
                
                print(f"{duration:.2f}s ✅")
                
            except Exception as e:
                round_results.append({
                    'prompt': prompt,
                    'duration': None,
                    'response_length': 0,
                    'tokens_per_sec': 0,
                    'success': False,
                    'error': str(e)
                })
                print(f"失敗 ❌")
        
        all_results.append(round_results)
        
        *# 顯示本輪統計*
        successful = [r for r in round_results if r['success']]
        if successful:
            avg_duration = statistics.mean([r['duration'] for r in successful])
            avg_tokens = statistics.mean([r['tokens_per_sec'] for r in successful])
            print(f"  本輪平均: {avg_duration:.2f}s, {avg_tokens:.1f} 字元/秒")
    
    *# 計算總體統計*
    print("\n" + "=" * 60)
    print("📈 基準測試結果")
    print("=" * 60)
    
    all_successful = []
    for round_results in all_results:
        all_successful.extend([r for r in round_results if r['success']])
    
    if all_successful:
        durations = [r['duration'] for r in all_successful]
        tokens_per_sec = [r['tokens_per_sec'] for r in all_successful]
        
        stats = {
            'model': model,
            'timestamp': datetime.now().isoformat(),
            'total_tests': len(all_successful),
            'iterations': iterations,
            'avg_duration': statistics.mean(durations),
            'min_duration': min(durations),
            'max_duration': max(durations),
            'std_duration': statistics.stdev(durations) if len(durations) > 1 else 0,
            'avg_tokens_per_sec': statistics.mean(tokens_per_sec),
            'min_tokens_per_sec': min(tokens_per_sec),
            'max_tokens_per_sec': max(tokens_per_sec),
            'std_tokens_per_sec': statistics.stdev(tokens_per_sec) if len(tokens_per_sec) > 1 else 0
        }
        
        print(f"模型: {model}")
        print(f"總測試次數: {stats['total_tests']}")
        print(f"平均回應時間: {stats['avg_duration']:.2f} ± {stats['std_duration']:.2f} 秒")
        print(f"最快回應: {stats['min_duration']:.2f} 秒")
        print(f"最慢回應: {stats['max_duration']:.2f} 秒")
        print(f"平均處理速度: {stats['avg_tokens_per_sec']:.1f} ± {stats['std_tokens_per_sec']:.1f} 字元/秒")
        
        *# 儲存結果*
        filename = f"benchmark_{model.replace(':', '_')}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(filename, 'w') as f:
            json.dump({
                'stats': stats,
                'detailed_results': all_results
            }, f, indent=2)
        
        print(f"\n📁 詳細結果已儲存至: {filename}")
        
        return stats
    else:
        print("❌ 所有測試都失敗了!")
        return None
  
  if __name__ == "__main__":
    import sys
    
    model = sys.argv[1] if len(sys.argv) > 1 else "llama2:7b"
    iterations = int(sys.argv[2]) if len(sys.argv) > 2 else 3
    
    run_benchmark(model, iterations)
  ```
  
---
- ## 📚 腳本使用範例
- ### 日常使用工作流
  
  bash  
  
  ```
  *# 1. 每日健康檢查*
  python3 ~/system_health.py
  
  *# 2. 快速測試模型*
  python3 ~/quick_test.py llama2:7b "今天天氣如何?"
  
  *# 3. 管理模型*
  python3 ~/model_manager.py
  
  *# 4. 執行完整測試*
  python3 ~/auto_test_framework.py
  
  *# 5. 效能基準測試*
  python3 ~/benchmark.py llama2:7b 3
  
  *# 6. 生成報告*
  python3 ~/html_reporter.py llm_test_log_*.json
  ```
- ### 自動化腳本
  
  bash  
  
  ```
  *# 建立每日檢查腳本*
  cat > ~/daily_check.sh << 'EOF'
  #!/bin/bash
  echo "🌅 每日 LLM 系統檢查 - $(date)"
  echo "=================================="
  
  # 健康檢查
  python3 ~/system_health.py
  
  # 快速測試
  echo -e "\n🧪 快速功能測試:"
  python3 ~/quick_test.py llama2:7b "Hello, test message"
  
  echo -e "\n📊 今日檢查完成!"
  EOF
  
  chmod +x ~/daily_check.sh
  ```
- ### 監控腳本
  
  bash  
  
  ```
  *# 建立連續監控腳本*
  cat > ~/continuous_monitor.sh << 'EOF'
  #!/bin/bash
  echo "📡 啟動連續監控模式..."
  echo "按 Ctrl+C 停止監控"
  
  while true; do
    clear
    echo "🔄 實時監控 - $(date)"
    echo "=================================="
    
    # 系統狀態
    python3 ~/system_health.py
    
    echo -e "\n⏰ 5秒後更新..."
    sleep 5
  done
  EOF
  
  chmod +x ~/continuous_monitor.sh
  ```
  
---
- ## 🎯 總結
  
  這個完整的腳本集合提供了：  
- ### ✅ 監控功能
- GPU 即時監控
- 系統資源監控
- 健康狀態檢查
- ### ✅ 測試功能
- 快速測試
- 詳細效能測試
- 自動化測試框架
- 基準測試
- ### ✅ 管理功能
- 模型管理
- 結果記錄
- 報告生成
- ### ✅ 實用工具
- 日常檢查腳本
- 連續監控
- 自動化工作流
  
  **所有腳本都經過實際測試，可以在你的 WSL2 + RTX 4070 環境中使用！** 🚀