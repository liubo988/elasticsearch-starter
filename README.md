# elasticsearch-starter
es-7.5版本的封装starter组件。
================================================================================================================================================================    
1.首先在pom文件中引入组件的依赖：

<!--elasticsearch-starter 配置依赖开始 -->
			<dependency>
				<groupId>com.ygw</groupId>
				<artifactId>zeus-elasticsearch-starter</artifactId>
				<!--排除这个slf4j-log4j12 -->
				<exclusions>
					<exclusion>
						<groupId>org.slf4j</groupId>
						<artifactId>slf4j-log4j12</artifactId>
					</exclusion>
				</exclusions>
				<version>${zeus-elasticsearch.version}</version>
			</dependency>
			<!--elasticsearch-starter 配置依赖结束 -->
			
================================================================================================================================================================    	
配置文件添加如下：
ygw:
  elasticsearch:
    nodes: 172.19.150.71:9200,172.19.53.37:9200,172.19.208.183:9200
    client:
      connect-timeout: 1500
      connection-request-timeout: 1000
      socket-timeout: 1500
      max-conn-total: 2000
      max-conn-per-route: 500
    log:
      enable: true
      
================================================================================================================================================================    
2.项目的启动类里面排除es的官方依赖：

@EnableAutoConfiguration(exclude = {RestClientAutoConfiguration.class})

================================================================================================================================================================    
3.在需要的controller或service方法里面引入封装的esClient:

 @Autowired
 private ElasticsearchClient client;
 
 package com.ygw.luban.controller.common;

import com.alibaba.fastjson.JSONArray;

import com.alibaba.fastjson.JSONObject;
import com.ygw.es.client.ElasticsearchClient;
import com.ygw.es.core.UUIDUtils;
import com.ygw.luban.common.constant.ESIndex;
import com.ygw.luban.common.dto.StudentCourseArrangeDto;
import com.ygw.luban.common.dto.es.StudentAnswersDto;
import com.ygw.luban.common.page.PageQuery;
import com.ygw.luban.common.pojo.es.LearningReport;
import com.ygw.luban.common.pojo.es.StudentAnswers;
import com.ygw.luban.common.pojo.mongo.QuestionConnectionCell;
import com.ygw.luban.common.pojo.mongo.QuestionOption;
import com.ygw.luban.service.es.IStudentAnswerESService;
import com.ygw.utils.DateUtil;
import com.ygw.utils.Response;
import com.ygw.utils.constants.HttpCode;
import com.ygw.utils.token.Luna;
import io.seata.spring.annotation.GlobalTransactional;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;

@Api(value = "TestESController", tags = "")
@RestController
@RequestMapping(value = "/testES")
public class TestESController {

    @Autowired
    private IStudentAnswerESService studentAnswersService;

    @Autowired
    private ElasticsearchClient client;
 
    @ApiOperation(value = "测试创建学生答案索引")
    @PostMapping("/testStudentAnswerIndex")
    @ResponseBody
    @Luna
    public Response<String> testStudentAnswerIndex(HttpServletRequest request) throws Exception {
        client.indices().create(ESIndex.STUDENT_ANSWERS.value(), new StudentAnswers());
        return new Response<String>("成功", HttpCode.OK.msg(), HttpCode.OK);
    }
    
    
    @ApiOperation(value = "测试查询")
    @GetMapping("/testQuery")
    @ResponseBody
    @Luna
    public Response<String> testQuery() throws Exception {
        StudentAnswersDto sa = new StudentAnswersDto();
        PageQuery pageQuery = new PageQuery();
        pageQuery.setPageSize(10);
        pageQuery.setCurrPage(2);
        List<StudentAnswersDto> studentAnswersDtoList = studentAnswersService.findPageListByParams(new StudentAnswersDto(), pageQuery);
        return new Response<String>("成功", HttpCode.OK.msg(), HttpCode.OK);
    }


    @ApiOperation(value = "测试插入数据")
    @PostMapping("/testInsert")
    @ResponseBody
    @Luna
    public Response<String> testInsert(HttpServletRequest request) throws Exception {
        StudentAnswersDto sa = new StudentAnswersDto();
        sa.setCreateTime(System.currentTimeMillis());
        sa.setMasterTeacherName("小王");
        sa.setStat(1);
        sa.setAnswerResult(1);
        sa.setIsAnswer(2);
        List<StudentAnswersDto> dtoList = studentAnswersService.findPageListByParams(new StudentAnswersDto(), null);
        sa.setStudentAnswerMarkTime(dtoList.get(0).getStudentAnswerMarkTime());
        sa.setStudentAnswerSubmitTime(new Date());
        sa.setCreateTimeStr(DateUtil.getDateStr(new Date()));
        List<List<QuestionConnectionCell>> qcLists = new ArrayList<>();
        List<QuestionConnectionCell> qcList = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            QuestionConnectionCell qc = new QuestionConnectionCell();
            qc.setNo(String.valueOf(i));
            qc.setPic("img" + i);
            qc.setType(String.valueOf(i));
            qc.setText("选项" + i);
            qcList.add(qc);
        }
        qcLists.add(qcList);
        sa.setCellList(qcLists);
        List<QuestionOption> optionList = new ArrayList<>();
        int count = studentAnswersService.findCount(new StudentAnswersDto());
        sa.setClassId(String.valueOf(count + 1));
        sa.setClassName("测试" + count + 1);

        for (int i = 0; i < 5; i++) {
            QuestionOption questionOption = new QuestionOption();
            questionOption.setNo(count + String.valueOf(i));
            questionOption.setChoiceNumber(count + i);
            questionOption.setText("选项" + count + i);
            optionList.add(questionOption);
        }
        sa.setOptionList(optionList);
        studentAnswersService.insert(sa);
        return new Response<String>("成功", HttpCode.OK.msg(), HttpCode.OK);
    }

    @ApiOperation(value = "删除索引")
    @PostMapping("/deleteIndex")
    @ResponseBody
    @Luna
    public Response<String> deleteIndex() throws Exception {
        StudentAnswersDto sa = new StudentAnswersDto();
        studentAnswersService.deleteIndex();
        return new Response<String>("成功", HttpCode.OK.msg(), HttpCode.OK);
    }

}
    
