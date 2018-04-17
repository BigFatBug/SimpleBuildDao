<template>
  <div class="hello">
    <el-row>
      <el-col :span="2">
        <el-button type="success" round @click="buildDao">赐予我无限爱与被爱的力量</el-button>

        <div></div>
        <img src="../assets/test.gif" style="margin-top: 50px">
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
          '    @SQL("SELECT " + FIELD + " FROM " + TABLE_NAME + " WHERE mifi_id = :1 ")\n' +
          '    ${className} getByMifiId(long mifiId);\n' +
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
            res += fieldList[i] + '=:1.' + this.transfer(fieldList[i])
          }
        }
        return res
      },
      buildDao() {
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
        this.className = this.transfer(this.tableName)
        this.className = this.className[0].toUpperCase() + this.className.slice(1)
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
