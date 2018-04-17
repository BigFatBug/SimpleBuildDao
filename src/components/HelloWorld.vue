<template>
  <div class="hello">
    <el-alert
    title="表名根据CREATE TABLE那句话从正则里拿的"
    type="success">
  </el-alert>
    <el-alert
      title="字段是判断每行第一个非空字符是不是`"
      type="info">
    </el-alert>
    <el-alert
      title="所以字段啊表名啊都得用`包起来"
      type="warning">
    </el-alert>
    <el-alert
      title="java里面的包名啊引用啊就得自己加了"
      type="error">
    </el-alert>
    <el-row style="margin-top: 50px">
      <el-col :span="2">
        <el-button type="success" round @click="wakaka" style="margin-bottom: 50px">赐予我无限爱与被爱的力量</el-button>

        <div v-show="i>0">模块初始化...</div>
        <div v-show="i>1">正在加载...</div>
        <div v-show="i>2">正在引入流量...</div>
        <div v-show="i>3">打开监控窗...</div>
        <div v-show="i>4">开启discovery...</div>
        <div v-show="i>5">建造窗口环境...</div>
        <div v-show="i>6">模拟哈弗曼...</div>
        <div v-show="i>7">好吧上面都是骗人的</div>
        <img v-show="i>8" src="../assets/test.gif">
      </el-col>
      <el-col :span="4">
        建表语句：
      </el-col>
      <el-col :span="6">
        <el-input
          type="textarea"
          autosize
          :rows="2"
          placeholder="请输入建表语句"
          v-model="input">
        </el-input>
      </el-col>
      <el-col :span="4">
        转换结果：
      </el-col>
      <el-col :span="6">
        <el-input
          type="textarea"
          autosize
          :rows="2"
          placeholder="这里是结果"
          v-model="output">
        </el-input>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        msg: 'Welcome to Your Vue.js App',
        i: 0,
        interval: null,
        input: '',
        output: '',
        tableName: '',
        className: '',
        tableFieldList: [],
        classFieldList: [],
        classRowFunc: new Function('tableName', 'fieldStr', 'insertStr', 'updateStr', 'className', 'objectName', 'return' + '`import net.paoding.rose.jade.annotation.DAO;\n' +
          'import net.paoding.rose.jade.annotation.SQL;\n' +
          '\n' +
          '/**\n' +
          ' * Created by *** on ' + (new Date().getFullYear()) + '/' + (new Date().getMonth()) + '/' + (new Date().getDate()) + '.\n' +
          ' */\n' +
          '@DAO\n' +
          'public interface ${className}DAO {\n' +
          '    String TABLE_NAME = "${tableName}";\n' +
          '    String FIELD = " ${fieldStr}";\n' +
          '    String INSERT_VALUES = " NULL, ${insertStr} ";\n' +
          '    String UPDATE_COLUMNS = " ${updateStr}";\n' +
          '\n' +
          '    @SQL("INSERT INTO " + TABLE_NAME + "(" + FIELD + ")"\n' +
          '            + "VALUES(" + INSERT_VALUES + ")")\n' +
          '    int insert(${className} ${objectName});\n' +
          '\n' +
          '    @SQL("UPDATE " + TABLE_NAME + " SET " + UPDATE_COLUMNS + " WHERE id = :1.id")\n' +
          '    int update(${className} ${objectName});\n' +
          '\n' +
          '    @SQL("SELECT " + FIELD + " FROM " + TABLE_NAME + " WHERE id = :1 ")\n' +
          '    ${className} getById(long Id);\n' +
          '\n' +
          '    @SQL("DELETE FROM " + TABLE_NAME + " WHERE id = :1 ")\n' +
          '    long deleteById(long id);\n' +
          '}`')
      }
    },

    mounted() {

    },

    methods: {
      transfer(data) {
        return data.replace(/_./g, function (x) {
          return x.slice(1).toUpperCase();
        });
      },

      alert(msg, type) {
        this.$message({
          message: msg,
          type: type
        });
      },
      buildUpdateStr(fieldList) {
        let res = ''
        for (let i in fieldList) {
          if (fieldList[i] == 'id') {
            continue
          } else {
            if (i > 0) {
              res += ', '
            }
            res += fieldList[i] + ' = :1.' + this.transfer(fieldList[i])
          }
        }
        return res
      },
      wakaka(){
        this.i = 0
        this.interval = setInterval(this.buildDao, 250)
      },
      buildDao() {
        if(this.i < 8){
          this.i += 1
          return
        } else{
          clearInterval(this.interval)
        }
        this.i += 1
        let inputList = this.input.split('\n')
        for (let row of inputList) {
          row.toUpperCase()
          if (row.toUpperCase().indexOf('CREATE TABLE') >= 0) {
            this.tableName = row.match('`(.*?)`')[1]
            if (this.tableName == null) {
              this.alert('表名为空', 'error')
              return
            }
          } else {
            for (let c of row) {
              if (c == ' ') {
                continue
              } else if (c == '`') {
                let field = row.match('`(.*?)`')[1]
                if (field == null || field == '') {
                  break
                }
                this.tableFieldList.push(field)
              } else {
                break
              }
            }
          }
        }
        this.objectName = this.transfer(this.tableName)
        this.className = this.objectName[0].toUpperCase() + this.objectName.slice(1)
        let fieldStr = this.tableFieldList.join(', ')
        let noIdTableFieldList = this.tableFieldList.filter((val) => {return val != 'id'})
        let insertStr = this.transfer(noIdTableFieldList.join(', '))
        let updateStr = this.buildUpdateStr(noIdTableFieldList)
        this.output = this.classRowFunc(this.tableName, fieldStr, insertStr, updateStr, this.className, this.objectName)
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
