<template>
  <div>
    <input type="text" v-model="question" placeholder="输入您的问题" style="width: 300px;" />
    <button @click="startChat" :disabled="isRequesting">发送</button>
    <div v-if="thinking" class="thinking">
      🤔 正在思考: {{ thinking }}
    </div>
    <div id="answer" v-html="answer"></div>
  </div>
</template>

<script>
export default {
  name: "sample",
  data() {
    return {
      question: "", // 用户输入的查询
      answer: "", // 逐步显示的答案
      thinking: "", // 显示正在思考的状态
      eventSource: null, // 用于 SSE 连接的 EventSource 实例
      isRequesting: false, // 用于控制是否可以发送请求
    };
  },
  methods: {
    startChat() {
      // 清理旧数据
      this.answer = '';
      this.thinking = '';
      
      // 如果没有输入问题，给出提示
      if (!this.question.trim()) {
        alert("请输入问题内容");
        return;
      }

      // 如果已经有请求正在进行，防止重复点击
      if (this.isRequesting) {
        return;
      }

      // 标记为正在请求
      this.isRequesting = true;

      // 关闭已有的 SSE 连接
      if (this.eventSource) {
        this.eventSource.close();
      }

      // 创建 SSE 连接
      const url = `http://10.1.12.12:5069/chat?question=${encodeURIComponent(this.question)}`;
      this.eventSource = new EventSource(url);

      // 处理接收到的消息
      this.eventSource.onmessage = (e) => {
        try {
          const data = JSON.parse(e.data);

          // 更新正在思考的状态
          if (data.thinking) {
            this.thinking = data.thinking;
          }

          // 增量更新回答内容
          if (data.answer) {
            this.answer += data.answer.slice(this.answer.length);
          }

          // 如果没有更多消息，说明请求完成
          if (data.complete) {
            this.isRequesting = false; // 重置请求状态
            this.eventSource.close(); // 关闭连接
          }
        } catch (err) {
          console.error('数据解析失败:', err);
          this.isRequesting = false; // 发生错误时，停止请求
        }
      };

      // 处理连接关闭事件
      this.eventSource.onerror = () => {
        this.isRequesting = false; // 连接出错时停止请求
        this.eventSource.close();
      };
    },
  },
};
</script>

  
  <style scoped>
    .thinking {
      color: #666;
      font-style: italic;
      margin: 10px 0;
    }
    #answer { 
      border: 1px solid #ccc; 
      padding: 15px; 
      min-height: 100px; 
      margin: 10px 0;
      white-space: pre-wrap;
    }
  </style>
  