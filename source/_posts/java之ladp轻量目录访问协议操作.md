---
title: java之ladp轻量目录访问协议操作
date: 2018-06-24 13:15:02
tags: [java,ladp]
---

定义：LDAP是基于TCP/IP协议的目录访问协议，是Internet上目录服务的通用访问协议。LDAP的出现简化了X.500目录的复杂度，降低了开发成本，是X.500标准的目录访问协议DAP的子集，同时也作为IETF的一个正式标准。LDAP的核心规范在RFC中都有定义，所有与LDAP相关的RFC都可以在LDAPman RFC网页中找到。

<!--more-->

# ldap介绍

## ldap有什么用

在当今的信息世界，网络为人们提供了丰富的资源。随着网络资源的日益丰富，迫切需要一种能有效管理资源信息并利于检索查询的服务技术。目录服务技术随之产生。
1.LDAP目录服务可以有效地解决众多网络服务的用户账户问题。
2.LDAP目录服务规定了统一的身份信息数据库、身份认证机制和接口，实现了资源和信息的统一管理，保证了数据的一致性和完整性。
3.LDAP目录服务是以树状的层次结构来描述数据信息的，此种模型适应了众多行业应用的业务组织结构。

## ldap的好处

LDAP服务器也是用来处理查询和更新LDAP目录的。换句话来说LDAP目录也是一种类型的数据库，但是不是关系型数据库。不象被设计成每分钟需要处理成百上千条数据变化的数据库，例如:在电子商务中经常用到的在线交易处理(OLTP)系统，LDAP主要是优化数据读取的性能。
LDAP最大的优势是:可以在任何计算机平台上，用很容易获得的而且数目不断增加的LDAP的客户端程序访问LDAP目录。而且也很容易定制应用程序为它加上LDAP的支持。

# Softerra LDAP Browser 4.5

Softerra LDAP Browser 4.5在我看来就是一个可视化工具，可以清晰的看到目录结构和数据，下面说明一下如何简单的使用：

## 下载

http://download.csdn.net/detail/fddqfddq/4656393

这里就不详细了解此软件的使用，感兴趣的朋友可以自行了解一下。

# JAVA实现ldap的操作

通过eclpse创建maven项目，引入需要的包和spring的常用包，主要jar包如下 (文档编写匆忙可能不全)

	spring-ldap-1.3.1.RELEASE-all.jar
	commons-lang-2.6.jar
	commons-logging-2.6.jar
	spring-beans-3.05.jar
	spring-core-3.05.jar
	spring-tx-3.05.jar

创建基础类baseldap


```java
package com.meixin.ldap.base;

import java.util.Hashtable;

import javax.naming.Context;
import javax.naming.directory.DirContext;
import javax.naming.directory.InitialDirContext;
/**
 *
 * @Description ldap基础类
 * @author guowenbo
 * @date 2016年3月15日 下午3:53:57
 */
public class BaseLdap {

	//上下文，可由此进行简单操作
	public static DirContext dc = null;

	static {
		Hashtable<String, String> env = new Hashtable<String, String>();
		String LDAP_URL = "********";
		String adminName = "*******";
		String adminPassword = "******";
		env.put(Context.INITIAL_CONTEXT_FACTORY,"com.sun.jndi.ldap.LdapCtxFactory");
		env.put(Context.PROVIDER_URL, LDAP_URL);
		env.put(Context.SECURITY_AUTHENTICATION, "simple");
		env.put(Context.SECURITY_PRINCIPAL, adminName);
		env.put(Context.SECURITY_CREDENTIALS, adminPassword);
		try {
			dc = new InitialDirContext(env);
			System.out.println("账户认证成功！");
		} catch (javax.naming.AuthenticationException e) {
			System.out.println("账户认证失败，账号或密码不正确！");
		} catch (Exception e) {
			System.out.println("账户认证出错：" + e);
		}
	}
}
```

然后大概了解一下，写一个简单的工具类继承baseldap：

```java
package com.meixin.ldap.utils;

import javax.naming.NamingEnumeration;
import javax.naming.NamingException;
import javax.naming.directory.Attribute;
import javax.naming.directory.Attributes;
import javax.naming.directory.BasicAttribute;
import javax.naming.directory.BasicAttributes;
import javax.naming.directory.DirContext;
import javax.naming.directory.ModificationItem;
import javax.naming.directory.SearchControls;
import javax.naming.directory.SearchResult;

import com.meixin.ldap.base.BaseLdap;

/**
 *
 * @Description 对ldap操作的简易工具类
 * @author guowenbo
 * @date 2016年3月15日 下午4:19:12
 */
public class LdapUtils extends BaseLdap{

	/**
	 *
	 * @Description 创建新用户
	 * @author guoewenbo
	 * @date 2016年3月15日 下午5:13:46
	 * @param rootElement 某节点路径（将在此节点下创建）
	 * @param newUserName 名称
	 */
	public void add(String rootElement,String newUserName) {
		try {
			BasicAttributes attrs = new BasicAttributes();
			//创建子节点值
			BasicAttribute objclassSet = new BasicAttribute("objectClass");
			objclassSet.add("memberOf");
			objclassSet.add("msExchVersion");
			attrs.put(objclassSet);
			attrs.put("ou", newUserName);
			dc.createSubcontext("ou=" + newUserName + "," + rootElement, attrs);
		} catch (Exception e) {
			System.out.println("新增失败:" + e);
		}
	}

	/**
	 *
	 * @Description 删除用户
	 * @author guowenbo
	 * @date 2016年3月15日 下午5:16:25
	 * @param dn 账户所在的域全路径
	 */
	public void delete(String dn) {
		try {
			dc.destroySubcontext(dn);
		} catch (Exception e) {
			System.out.println("删除失败:" + e);
		}
	}

	/**
	 *
	 * @Description 更改节点名称
	 * @author guowenbo
	 * @date 2016年3月15日 下午5:17:27
	 * @param oldDN 旧节点名称全路径
	 * @param newDN 新节点名称全路径
	 * @return
	 */
	public boolean renameEntry(String oldDN, String newDN) {
		try {
			dc.rename(oldDN, newDN);
			return true;
		} catch (NamingException e) {
			System.out.println("更新失败: " + e);
			return false;
		}
	}

	/**
	 *
	 * @Description 更新节点下的属性与值 (节点属性的增删改，可扩展为批量操作)
	 * @author guowenbo
	 * @date 2016年3月15日 下午5:24:06
	 * @param dn 需要更新的节点全路径
	 * @param type 节点更新类型（默认0：新增，1：修改，2：删除）
	 * @param attribute 更改属性
	 * @param value 属性值
	 * @return
	 */
	public boolean modifyInformation(String dn,int type,String attribute, String value) {
		try {
			ModificationItem[] mods = new ModificationItem[1];
			Attribute attr0 = new BasicAttribute(attribute, value);
			if(type==0){
				mods[0] = new ModificationItem(DirContext.ADD_ATTRIBUTE, attr0);
			}else if(type==1){
				mods[0] = new ModificationItem(DirContext.REPLACE_ATTRIBUTE, attr0);
			}else if(type==2){
				mods[0] = new ModificationItem(DirContext.REMOVE_ATTRIBUTE, attr0);
			}else{
				System.out.println("type数据有误！");
			}
			dc.modifyAttributes(dn, mods);
			return true;
		} catch (NamingException e) {
			System.out.println("更新失败: " + e);
			return false;
		}
	}

	/**
	 *
	 * @Description 关闭ldap连接
	 * @author guowenbo
	 * @date 2016年3月15日 下午5:18:41
	 */
	public void close() {
		if (dc != null) {
			try {
				dc.close();
				System.out.println("关闭ldap连接成功。。。");
			} catch (NamingException e) {
				System.out.println("关闭失败:" + e);
			}
		}
	}


	/**
	 *
	 * @Description 简单的条件查询
	 * @author guowenbo
	 * @date 2016年3月15日 下午4:19:27
	 * @param base 基础节点 （dc=meixin,dc=com）
	 * @param scope 查询范围（默认0：遍历，1：单层，2：本节点）
	 * @param resultAttrutes[] 指定查询属性（不需要指定则传递null：查询符合条件的所有属性；）
	 * @param searchFilter 查询条件（属性名=值，如："sAMAccountName=guowenbo" 查询所有sAMAccountName属性值为guowenbo的数据；"objectclass=*"默认查询全部。）
	 */
	@SuppressWarnings("rawtypes")
	public void Ldapbyuserinfo(String base,int scope,String resultAttrutes[],String searchFilter) {
		SearchControls searchCtls = new SearchControls();
		if(scope==0){
			//默认
			searchCtls.setSearchScope(SearchControls.SUBTREE_SCOPE);
		}else if(scope==1){
			searchCtls.setSearchScope(SearchControls.ONELEVEL_SCOPE);
		}else if(scope==2){
			searchCtls.setSearchScope(SearchControls.OBJECT_SCOPE);
		}else{
			System.out.println("查询范围数据不正确！");
			return;
		}
		int totalResults = 0;
		searchCtls.setReturningAttributes(resultAttrutes);
		try {
			NamingEnumeration answer = dc.search(base, searchFilter,
					searchCtls);
			if (answer == null || answer.equals(null)) {
				System.out.println("返回结果为空或查询数据不存在！");
				return;
			}
			System.out.println("查询成功！继续解析节点数据。。。");
			while (answer.hasMoreElements()) {
				SearchResult sr = (SearchResult) answer.next();
				System.out.println("已获取到信息：" + sr.getName());
				System.out.println("*****************进行遍历********************");
				Attributes Attrs = sr.getAttributes();
				if (Attrs != null) {
					try {
						for (NamingEnumeration ne = Attrs.getAll(); ne.hasMore();) {
							Attribute Attr = (Attribute) ne.next();
							System.out.println("属性名称(↓为值)："+ Attr.getID().toString());
							for (NamingEnumeration e = Attr.getAll(); e.hasMore(); totalResults++) {
								System.out.println(e.next().toString());
							}
						}
					} catch (NamingException e) {
						System.err.println("执行出错: " + e);
					}
				}
			}
			System.out.println("*****************遍历结束********************");
			System.out.println("属性总数: " + totalResults);
		} catch (Exception e) {
			System.err.println("执行出错: " + e);
		}
	}

}
```

测试一下看看：

```java
package com.meixin.ldap.test;

import org.junit.Test;

import com.meixin.ldap.utils.LdapUtils;

public class LdapTest {

	private static String base = "dc=meixin,dc=com";
	private static String filter = "sAMAccountName=guowenbo";
	private static String resultElements[] = null;
	private static int scope = 0;

	@Test
	public void testQuery(){
		LdapUtils test = new LdapUtils();
		resultElements = new String[]{"sAMAccountName","msExchVersion","memberOf"};
		test.Ldapbyuserinfo(base,scope,resultElements,filter);
		test.close();
	}
}
```

结果：

![ldap](/images/2018-06-24/ldap_1.png "result")