<template>
    <div class="container">
      <!-- 头部 -->
      <div class="app-header">
        <h1>
          星图遥感科研助手
          <span class="beta-tag">(内测版)</span>
        </h1>
      </div>
  
      <div class="main-wrapper">
        <div class="sidebar">
    <!-- 左侧侧边栏 -->
    <div class="sidebar">
      <button class="icon-btn" @click="toggleSwitchModelModal" title="切换模型">
        <i class="icon icon-switch">M</i>
      </button>
      <button class="icon-btn" @click="newChat" title="新建对话">
        <i class="icon icon-new-chat">N</i>
      </button>
      <button class="icon-btn" @click="toggleHistoryChatModal" title="历史对话">
        <i class="icon icon-history">H</i>
      </button>
    </div>

    <!-- 切换模型弹窗 -->
    <div class="modal" v-if="showSwitchModelModal" @click.self="toggleSwitchModelModal">
      <div class="modal-content">
        <span class="modal-close" @click="toggleSwitchModelModal">&times;</span>
        <h3>选择模型</h3>
        <!-- 模型选项 -->
        <ul>
          <li>模型 A</li>
          <li>模型 B</li>
          <li>模型 C</li>
        </ul>
      </div>
    </div>

    <!-- 历史对话弹窗 -->
    <div class="modal" v-if="showHistoryChatModal" @click.self="toggleHistoryChatModal">
      <div class="modal-content">
        <span class="modal-close" @click="toggleHistoryChatModal">&times;</span>
        <h3>历史对话</h3>
        <!-- 历史对话列表 -->
        <ul>
          <li>对话 1</li>
          <li>对话 2</li>
          <li>对话 3</li>
        </ul>
      </div>
    </div>
</div>
  
        <!-- 主聊天区域 -->
        <div class="chat-container">
          <div class="chat-messages" ref="chatMessages">
            <!-- 示例按钮区域 -->
            <div class="examples">
              <div
                class="example-btn"
                @click="fillExample('对北京市2024年建筑物分布密度进行统计分析，并给出城市规划建议')"
              >
                <div class="example-title">北京市建筑密度分析</div>
                <div class="example-text">
                  “对北京市2024年建筑物分布密度进行统计分析，并给出城市规划建议”
                </div>
              </div>
              <div
                class="example-btn"
                @click="fillExample('利用多光谱遥感数据识别长三角地区近五年植被变化情况')"
              >
                <div class="example-title">植被变化分析</div>
                <div class="example-text">
                  “利用多光谱遥感数据识别长三角地区近五年植被变化情况”
                </div>
              </div>
              <div
                class="example-btn"
                @click="fillExample('基于SAR影像的洪涝灾害应急区域提取')"
              >
                <div class="example-title">灾害应急分析</div>
                <div class="example-text">
                  “基于SAR影像的洪涝灾害应急区域提取”
                </div>
              </div>
              <div
                class="example-btn"
                @click="fillExample('请生成京津冀地区夜间灯光指数变化趋势图')"
              >
                <div class="example-title">灯光指数可视化</div>
                <div class="example-text">
                  “请生成京津冀地区夜间灯光指数变化趋势图”
                </div>
              </div>
            </div>
            <div
              v-for="(msg, index) in chatMessages"
              :key="index"
              class="message"
              :class="msg.type + '-message'"
            >
              <img :src="msg.avatar" class="avatar" />
              <!-- v-html 用于展示可能含有换行或 HTML 标签的内容 -->
              <div class="bubble" v-html="msg.content"></div>
            </div>
            
          </div>
  
          <div class="input-container">
            <div class="input-group">
              <textarea
                type="text"
                id="userInput"
                v-model="userInput"
                @input="autoResize"
                placeholder="输入科研问题..."
                @keyup.enter="sendMessage"
              ></textarea>
              
              <div class="mode-buttons">
                <button class="mode-btn" @click="activateDeepThink">深度思考模式</button>
                <button class="mode-btn" @click="activateWebSearch">联网搜索</button>
                <label class="upload-btn" title="上传文件，支持多种格式文本和图像文件">
                +
                <input type="file" hidden />
              </label>
              </div>
              <button @click="sendMessage">发送</button>
            </div>
          </div>
        </div>
  
        <!-- 右侧工具区 -->
        <div class="tools-panel">
          <div class="tool-tabs">
            <button
              class="tab-btn"
              :class="{ active: activeTool === '学术搜索' }"
              @click="activeTool = '学术搜索'"
            >
              学术搜索
            </button>
            <button
              class="tab-btn"
              :class="{ active: activeTool === '编程工具' }"
              @click="activeTool = '编程工具'"
            >
              编程工具
            </button>
            <button
              class="tab-btn"
              :class="{ active: activeTool === '数据工具' }"
              @click="activeTool = '数据工具'"
            >
              数据工具
            </button>
          </div>
          <div class="tool-content">
            <iframe src="https://sc.panda321.com/" style="width: 100%; height: 600px;" frameborder="0"></iframe>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: "ChatApp",
    data() {
      return {
        isProcessing: false,
        userInput: "",
        showSwitchModelModal: false,
        showHistoryChatModal: false,
        // 初始化时添加一条 AI 消息
        chatMessages: [
          {
            type: "ai",
            avatar: "/static/images/avatar-ai.png",
            content: `您好！我是星图遥感科研助手，可以为您提供以下专业支持：<br>
            🔍 遥感数据分析：多源数据融合处理、特征提取与解译<br>
            📊 统计图表生成：时空变化可视化、多维数据呈现<br>
            💡 科研建议支持：研究方案设计、方法优化指导<br>
            以下示例典型场景可以快速开始：`,
          },
        ],
        historyList: ["城市规划分析...", "植被变化研究...", "洪涝灾害评估..."],
        selectedModel: "deepseek R1",
        activeTool: "学术搜索",
      };
    },
    methods: {
        autoResize(event) {
      const textarea = event.target;
      // 重置高度，以便获取正确的 scrollHeight
      textarea.style.height = 'auto';
      // 定义最大高度（需与 CSS max-height 保持一致）
      const maxHeight = 400; // 对应5行文本的高度，可根据实际情况调整
      // 设置新的高度，不超过最大高度
      textarea.style.height = Math.min(textarea.scrollHeight, maxHeight) + 'px';
    },
        toggleSwitchModelModal() {
      this.showSwitchModelModal = !this.showSwitchModelModal;
      console.log(this.showSwitchModelModal);
    },
    toggleHistoryChatModal() {
      this.showHistoryChatModal = !this.showHistoryChatModal;
    },
    newChat() {
      // 新建对话逻辑处理
      console.log('新建对话');
    },
      fillExample(text) {
        this.userInput = text;
      },
      newChat() {
        // 重置对话记录，可根据需要保存历史记录
        this.chatMessages = [];
      },
      scrollToBottom() {
        this.$nextTick(() => {
          const container = this.$refs.chatMessages;
          if (container) {
            container.scrollTop = container.scrollHeight;
          }
        });
      },
      async sendMessage() {
        if (this.isProcessing || !this.userInput.trim()) return;
        const query = this.userInput.trim();
        // 清空输入框
        this.userInput = "";
        // 添加用户消息
        this.chatMessages.push({
          type: "user",
          avatar: "/static/images/avatar-user.png",
          content: query,
        });
        // 创建 AI 消息占位
        const aiMessage = {
          type: "ai",
          avatar: "/static/images/avatar-ai.png",
          content: "",
        };
        this.chatMessages.push(aiMessage);
        this.scrollToBottom();
  
        this.isProcessing = true;
        try {
          const response = await fetch("/api/chat", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ query }),
          });
          const reader = response.body.getReader();
          const decoder = new TextDecoder("utf-8");
          let buffer = "";
          while (true) {
            const { done, value } = await reader.read();
            if (done) break;
            buffer += decoder.decode(value, { stream: true });
            const events = buffer.split("\n\n");
            // 处理完整的数据块
            for (let i = 0; i < events.length - 1; i++) {
              const event = events[i].trim();
              if (!event.startsWith("data: ")) continue;
              try {
                const data = JSON.parse(event.slice(6));
                aiMessage.content += data.content;
                this.scrollToBottom();
              } catch (e) {
                console.error("Parse error:", e);
              }
            }
            buffer = events[events.length - 1];
          }
          // 处理剩余内容
          if (buffer) {
            try {
              const data = JSON.parse(buffer.slice(6));
              aiMessage.content += data.content;
            } catch (e) {
              console.error("Final parse error:", e);
            }
          }
          // 如果返回的内容包含步骤提示，可调用 processStepResponse
          if (aiMessage.content.includes("Step 1:")) {
            this.processStepResponse(aiMessage);
          }
        } catch (error) {
          console.error("Fetch error:", error);
          aiMessage.content = "请求失败，请重试";
        } finally {
          this.isProcessing = false;
        }
      },
      processStepResponse(aiMessage) {
        // 可在此解析步骤化响应，并将消息内容拆分为多个步骤显示。
        // 此处仅作示例处理，你可以根据具体需求将逻辑拆分为独立组件。
        console.log("Step response:", aiMessage.content);
      },
      activateDeepThink() {
        // 深度思考模式逻辑
        console.log("深度思考模式已激活");
      },
      activateWebSearch() {
        // 联网搜索模式逻辑
        console.log("联网搜索模式已激活");
      },
    },
  };
  

  import '../assets/css/style.css'
  </script>
  
  <style scoped>
</style>
  