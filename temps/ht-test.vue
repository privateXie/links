<template>
  <div class="search">
    <Card>
      <Form ref="form" :model="form" :label-width="120">
        <FormItem label="输入数据">
          <textarea
            v-model="form.inputData"
            class="custom-textarea"
            placeholder="请输入格式字符串，例如：102.134.49.165:6005:http | : | In/Out: Japan-Tokyo To Tokyo[住宅IP] | Status: ✅"
          ></textarea>
        </FormItem>
        <FormItem>
          <Button type="primary" @click="handleConvert">转换</Button>
          <Button style="margin-left: 10px" @click="handleClear">清空</Button>
        </FormItem>
      </Form>

      <div v-if="resultList.length > 0" class="result-section">
        <Divider>转换结果</Divider>
        
        <Card class="final-result-card">
          <div class="final-result-header">
            <span class="final-result-title">最终结果（合并）：</span>
            <Button type="primary" size="small" @click="handleCopyAll">复制全部</Button>
          </div>
          <Input
            v-model="finalResultText"
            type="textarea"
            :rows="resultList.length + 2"
            readonly
            class="final-result-textarea"
          />
        </Card>

        <Divider>详细信息</Divider>
        
        <div v-for="(item, index) in resultList" :key="index" class="result-item">
          <Card>
            <div class="result-content">
              <div class="result-label">原始数据：</div>
              <div class="result-value">{{ item.original }}</div>
            </div>
            <div class="result-content">
              <div class="result-label">提取的IP:端口：</div>
              <div class="result-value">{{ item.ipPort }}</div>
            </div>
            <div class="result-content">
              <div class="result-label">Base64编码：</div>
              <div class="result-value">{{ item.base64 }}</div>
            </div>
            <div class="result-content">
              <div class="result-label">提取的地址信息：</div>
              <div class="result-value">{{ item.location }}</div>
            </div>
            <div class="result-content">
              <div class="result-label">最终结果：</div>
              <div class="result-value final-result">{{ item.final }}</div>
            </div>
            <div class="result-actions">
              <Button type="primary" size="small" @click="handleCopy(item.final)">复制</Button>
            </div>
          </Card>
        </div>
      </div>
    </Card>
  </div>
</template>

<script>
export default {
  name: "wapList",
  data() {
    return {
      form: {
        inputData: ""
      },
      resultList: []
    };
  },
  computed: {
    finalResultText() {
      return this.resultList.map(item => item.final).join("\n");
    }
  },
  methods: {
    handleConvert() {
      if (!this.form.inputData.trim()) {
        this.$Message.warning("请输入数据");
        return;
      }

      const lines = this.form.inputData.split("\n").filter(line => line.trim());
      this.resultList = [];

      lines.forEach(line => {
        const result = this.parseAndConvert(line.trim());
        if (result) {
          this.resultList.push(result);
        }
      });

      if (this.resultList.length === 0) {
        this.$Message.error("数据格式不正确，请检查输入");
      }
    },

    parseAndConvert(line) {
      try {
        const parts = line.split("|").map(p => p.trim());
        
        if (parts.length < 3) {
          return null;
        }

        const ipPortPart = parts[0].split(":");
        if (ipPortPart.length < 2) {
          return null;
        }

        const ipPort = `${ipPortPart[0]}:${ipPortPart[1]}`;
        
        const locationMatch = line.match(/In\/Out:\s*(.+?)\s*\|/);
        const location = locationMatch ? locationMatch[1] : "";

        const base64 = this.base64Encode(ipPort);
        const final = `http://${base64}?method=auto#${location}`;

        return {
          original: line,
          ipPort: ipPort,
          base64: base64,
          location: location,
          final: final
        };
      } catch (error) {
        console.error("解析错误:", error);
        return null;
      }
    },

    base64Encode(str) {
      try {
        return btoa(unescape(encodeURIComponent(str)));
      } catch (error) {
        console.error("Base64编码错误:", error);
        return "";
      }
    },

    handleCopy(text) {
      const textarea = document.createElement("textarea");
      textarea.value = text;
      document.body.appendChild(textarea);
      textarea.select();
      try {
        document.execCommand("copy");
        this.$Message.success("复制成功");
      } catch (error) {
        this.$Message.error("复制失败");
      }
      document.body.removeChild(textarea);
    },

    handleCopyAll() {
      const text = this.finalResultText;
      const textarea = document.createElement("textarea");
      textarea.value = text;
      document.body.appendChild(textarea);
      textarea.select();
      try {
        document.execCommand("copy");
        this.$Message.success("复制全部成功");
      } catch (error) {
        this.$Message.error("复制失败");
      }
      document.body.removeChild(textarea);
    },

    handleClear() {
      this.form.inputData = "";
      this.resultList = [];
    }
  }
};
</script>

<style scoped lang="scss">
.search {
  padding: 20px;
}

.custom-textarea {
  width: 100%;
  min-height: 150px;
  padding: 10px;
  border: 1px solid #dcdee2;
  border-radius: 4px;
  font-size: 14px;
  line-height: 1.5;
  resize: vertical;
  outline: none;
  transition: border-color 0.2s;
  
  &:focus {
    border-color: #57a3f3;
  }
  
  &::placeholder {
    color: #c5c8ce;
  }
}

.result-section {
  margin-top: 20px;
}

.final-result-card {
  margin-bottom: 20px;
  background: #e6f7ff;
  border: 1px solid #91d5ff;
}

.final-result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.final-result-title {
  font-weight: bold;
  color: #1890ff;
  font-size: 16px;
}

.final-result-textarea {
  background: #fff;
  border: 1px solid #91d5ff;
  
  ::v-deep textarea {
    font-family: monospace;
    font-size: 14px;
    color: #1890ff;
    line-height: 1.6;
  }
}

.result-item {
  margin-bottom: 20px;
}

.result-content {
  margin-bottom: 10px;
  
  &:last-child {
    margin-bottom: 0;
  }
}

.result-label {
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}

.result-value {
  background: #f5f7fa;
  padding: 8px 12px;
  border-radius: 4px;
  word-break: break-all;
  font-family: monospace;
  font-size: 14px;
  color: #606266;
}

.final-result {
  background: #e6f7ff;
  border: 1px solid #91d5ff;
  color: #1890ff;
  font-weight: 500;
}

.result-actions {
  margin-top: 15px;
  text-align: right;
}
</style>
