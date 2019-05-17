## 前提：
1、首先升级newbie版本为当前最新版，JPQL和普通的SQL没有语法区别，普通SQL仅限于特定业务。

2、在`resources`目录下新建目录`sqls`，如果存在则不需要新建。

## 测试场景：根据组织机构中的五表联查调职人员信息列表来查询数据

1、在`sqls`目录下新建文件，如`nativesql.sftl`，注意文件名必须以`.sftl`结尾。
sql格式如下：

```
#获取调职人员信息列表
-- getDZRYXXLB
  SELECT
	rydz.dzbh,rydz.rybm,rybm.mc,rybm.sfzh,rydz.dwbm,dwbm.dwmc,
    rydz.dzdwbm,dw.dwmc dzdwmc,rydz.czrmc,rydz.jsrmc,rydz.jszt
  FROM
	ZZJG_YX_RYDZ rydz
	INNER JOIN ZZJG_XT_DWRY dwry ON rydz.dwbm = dwry.dwbm
	AND rydz.rybm = dwry.rybm AND rydz.sfsc = 'N' AND dwry.sfsc = 'N'
	INNER JOIN ZZJG_XT_RYBM rybm ON rydz.rybm = rybm.rybm AND rybm.sfsc = 'N'
	INNER JOIN ZZJG_XT_DWBM dwbm ON rydz.dwbm = dwbm.dwbm
	INNER JOIN ZZJG_XT_DWBM dw ON rydz.dzdwbm = dw.dwbm
  WHERE

#调职查询为1的情况
  <#if dzcx == '0'>
    rydz.dwbm = :dwbm
      <#if dzdwbm??>
        AND rydz.dzdwbm = :dzdwbm
      <#else>
        OR rydz.dzdwbm =:dwbm
      </#if>
  </#if>

  <#if dzcx == '1'>
      rydz.dwbm =:dwbm
  </#if>
  <#if dzcx == '2'>
      rydz.dwbm = :dwbm
        <#if dzdwbm != ''>
           AND rydz.dzdwbm =:dzdwbm
        </#if>
  </#if>
 <#if zt != '0'>
 AND rydz.jszt =:zt
 </#if>

```
备注：sql必须以`-- `开始，然后加上sql名称，如需要给代码添加注释，需新起一行用`#`开头，然后添加具体注释即可。变量必须以
`:`开头，判断属性等于某个常量的时候使用`==`符号，其他具体代码请根据具体业务逻辑处理。

2、仓库中有以下方法
```
/**
  * 按照JPQL模板查询
  * @param entity
  * @return
  */
<S> List<S> queryWithTemplate(String tplName, String method, Object entity, Class<S> resultType);

/**
  * 按照JPQL模板分页查询
  * @param entity
  * @return
  */
<S> Pagination<S> queryPageWithTemplate(String tplName, String method,  Object entity,Class<S> resultType, PageRequest pageRequest);

/**
  * 按照原生SQL模板查询
  * @param entity
  * @return
  */
  <S> List<S> nativeQueryWithTemplate( String tplName, String method, Object entity,Class<S> resultType);

/**
  * 按照原生SQL模板分页查询
  * @param entity
  * @return
  */
  <S> Pagination<S> nativeQueryWithTemplate(String tplName, String method, Object entity,Class<S> resultType, PageRequest pageRequest);
```
