# 生成器模式

标签 ： 设计模式

---

## 初识生成器模式
###定义

> 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

### 结构和说明

![image_1cf2nei5jou68e915eg13oouvk9.png-51.3kB][1]

> Builder：生成器接口，定义创建一个Product对象所需的各个部件的操作。
> ConcreteBuilder：具体的生成器实现，实现各个部件的创建，并负责组装Product对象的各个部件，同时还提供一个让用户获取组装完成后的产品对象的方法。
> Director：指导者，也被称为导向者，主要用来使用Builder接口，以一个统一的过程来构建所需要的Product对象。
> Product：产品，表示被生成器构建的复杂对象，包含多个部件。

### 代码

```
/**
 * 被构建的产品对象的接口
 */
public interface Product {
	//定义产品的操作
}

/**
 * 构建器接口，定义创建一个产品对象所需的各个部件的操作
 */
public interface Builder {
	/**
	 * 示意方法，构建某个部件
	 */
	public void buildPart();
}
/**
 * 具体的构建器实现对象
 */
public class ConcreteBuilder implements Builder {
	/**
	 * 构建器最终构建的产品对象
	 */
	private Product resultProduct;
	/**
	 * 获取构建器最终构建的产品对象
	 * @return 构建器最终构建的产品对象
	 */
	public Product getResult() {
		return resultProduct;
	}
	public void buildPart() {
		//构建某个部件的功能处理
	}
}
/**
 * 指导者，指导使用构建器的接口来构建产品的对象
 */
public class Director {
	/**
	 * 持有当前需要使用的构建器对象
	 */
	private Builder builder;
	/**
	 * 构造方法，传入构建器对象
	 * @param builder 构建器对象
	 */
	public Director(Builder builder) {
		this.builder = builder;
	}
	/**
	 * 示意方法，指导构建器构建最终的产品对象
	 */
	public void construct() {
		//通过使用构建器接口来构建最终的产品对象
		builder.buildPart();
	}
}
```

> 请参考github designpattern项目下builder下example2包中的的代码实例

 
## 体会生成器模式
### 实现导出数据的应用框架

对于导出数据的应用框架，通常对于具体的导出内容和格式是有要求的，简单描述一下：
（1）导出的文件，不管什么格式，都分成三个部分，分别是文件头、文件体和文件尾
（2）在文件头部分，需要描述如下信息：分公司或门市点编号、导出数据的日期，对于文本格式，中间用逗号分隔
（3）在文件体部分，需要描述如下信息：表名称、然后分条描述数据。对于文本格式，表名称单独占一行，数据描述一行算一条数据，字段间用逗号分隔。
（4）在文件尾部分，需要描述如下信息：输出人
 
现在就要来实现上述功能。这里只关心如何实现导出文件，而且只实现导出成文本格式和XML格式就可以了，其它的就不去考虑了。
 
### 不用模式的解决方案

```
/**
 * 描述输出到文件头的内容的对象
 */
public class ExportHeaderModel {
	/**
	 * 分公司或门市点编号
	 */
	private String depId;
	/**
	 * 导出数据的日期
	 */
	private String exportDate;
}

/**
 * 描述输出数据的对象
 */
public class ExportDataModel {
	/**
	 * 产品编号
	 */
	private String productId;
	/**
	 * 销售价格
	 */
	private double price;
	/**
	 * 销售数量
	 */
	private double amount;
}

/**
 * 描述输出到文件尾的内容的对象
 */
public class ExportFooterModel {
	/**
	 * 输出人
	 */
	private String exportUser;
}

/**
 * 导出数据到文本文件的对象
 */
public class ExportToTxt {
	/**
	 * 导出数据到文本文件
	 * @param ehm 文件头的内容
	 * @param mapData 数据的内容
	 * @param efm 文件尾的内容
	 */
	public void export(ExportHeaderModel ehm,Map<String,Collection<ExportDataModel>> mapData,ExportFooterModel efm){
		//用来记录最终输出的文件内容
		StringBuffer buffer = new StringBuffer();
		//1：先来拼接文件头的内容
		buffer.append(ehm.getDepId()+","+ehm.getExportDate()+"\n");
		//2：接着来拼接文件体的内容
		for(String tblName : mapData.keySet()){
			//先拼接表名称
			buffer.append(tblName+"\n");
			//然后循环拼接具体数据
			for(ExportDataModel edm : mapData.get(tblName)){
				buffer.append(edm.getProductId()+","+edm.getPrice()+","+edm.getAmount()+"\n");
			}
		}
		//3：接着来拼接文件尾的内容
		buffer.append(efm.getExportUser());
		
		//为了演示简洁性，这里就不去写输出文件的代码了
		//把要输出的内容输出到控制台看看
		System.out.println("输出到文本文件的内容：\n"+buffer);
	}
}

/**
 * 导出数据到XML文件的对象
 */
public class ExportToXml {
	/**
	 * 导出数据到XML文件
	 * @param ehm 文件头的内容
	 * @param mapData 数据的内容
	 * @param efm 文件尾的内容
	 */
	public void export(ExportHeaderModel ehm,Map<String,Collection<ExportDataModel>> mapData,ExportFooterModel efm){
		//用来记录最终输出的文件内容
		StringBuffer buffer = new StringBuffer();
		//1：先来拼接文件头的内容
		buffer.append("<?xml version='1.0' encoding='gb2312'?>\n");
		buffer.append("<Report>\n");
		buffer.append("  <Header>\n");
		buffer.append("    <DepId>"+ehm.getDepId()+"</DepId>\n");
		buffer.append("    <ExportDate>"+ehm.getExportDate()+"</ExportDate>\n");
		buffer.append("  </Header>\n");
		//2：接着来拼接文件体的内容
		buffer.append("  <Body>\n");
		for(String tblName : mapData.keySet()){
			//先拼接表名称
			buffer.append("    <Datas TableName=\""+tblName+"\">\n");
			//然后循环拼接具体数据
			for(ExportDataModel edm : mapData.get(tblName)){
				buffer.append("      <Data>\n");
				buffer.append("        <ProductId>"+edm.getProductId()+"</ProductId>\n");
				buffer.append("        <Price>"+edm.getPrice()+"</Price>\n");
				buffer.append("        <Amount>"+edm.getAmount()+"</Amount>\n");
				buffer.append("      </Data>\n");
			}
			buffer.append("    </Datas>\n");
		}
		buffer.append("  </Body>\n");
		//3：接着来拼接文件尾的内容
		buffer.append("  <Footer>\n");
		buffer.append("    <ExportUser>"+efm.getExportUser()+"</ExportUser>\n");
		buffer.append("  </Footer>\n");
		buffer.append("</Report>\n");
		
		//为了演示简洁性，这里就不去写输出文件的代码了
		//把要输出的内容输出到控制台看看
		System.out.println("输出到XML文件的内容：\n"+buffer);
	}
}

public class Client {
	public static void main(String[] args) {
		//准备测试数据
		ExportHeaderModel ehm = new ExportHeaderModel();
		ehm.setDepId("一分公司");
		ehm.setExportDate("2010-05-18");
		
		Map<String,Collection<ExportDataModel>> mapData = new HashMap<String,Collection<ExportDataModel>>();
		Collection<ExportDataModel> col = new ArrayList<ExportDataModel>();
		
		ExportDataModel edm1 = new ExportDataModel();
		edm1.setProductId("产品001号");
		edm1.setPrice(100);
		edm1.setAmount(80);
		
		ExportDataModel edm2 = new ExportDataModel();
		edm2.setProductId("产品002号");
		edm2.setPrice(99);
		edm2.setAmount(55);		
		//把数据组装起来
		col.add(edm1);
		col.add(edm2);		
		mapData.put("销售记录表", col);
		
		ExportFooterModel efm = new ExportFooterModel();
		efm.setExportUser("张三");
		
		//测试输出到文本文件
		ExportToTxt toTxt = new ExportToTxt();
		toTxt.export(ehm, mapData, efm);
		//测试输出到xml文件
		ExportToXml toXml = new ExportToXml();
		toXml.export(ehm, mapData, efm);
		
	}
}
```

> 请参考github designpattern项目下builder下example1包中的代码示例

存在的问题是对于不同的输出格式，处理步骤是一样的，但是具体每步的实现是不一样的。
（1）先拼接文件头的内容
（2）然后拼接文件体的内容
（3）再拼接文件尾的内容
（4）最后把拼接好的内容输出出去成为文件

按照现在的实现方式，就存在如下的问题：
（1）构建每种输出格式的文件内容的时候，都会重复这几个处理步骤，应该提炼出来，形成公共的处理过程
（2）今后可能会有很多不同输出格式的要求，这就需要在处理过程不变的情况下，能方便的切换不同的输出格式的处理
换句话说，也就是构建每种格式的数据文件的处理过程，应该和具体的步骤实现分开，这样就能够复用处理过程，而且能很容易的切换不同的输出格式。
 
### 使用模式的解决方案

![image_1cf6rul0pvh91lsqaaf15a210169.png-69.8kB][2]

```
/**
 * 构建器接口，定义创建一个输出文件对象所需的各个部件的操作
 */
public interface Builder {
	/**
	 * 构建输出文件的Header部分
	 * @param ehm 文件头的内容
	 */
	public void buildHeader(ExportHeaderModel ehm);
	/**
	 * 构建输出文件的Body部分
	 * @param mapData 要输出的数据的内容
	 */
	public void buildBody(Map<String, Collection<ExportDataModel>> mapData);
	/**
	 * 构建输出文件的Footer部分
	 * @param efm 文件尾的内容
	 */
	public void buildFooter(ExportFooterModel efm);
}

/**
 * 实现导出数据到文本文件的的构建器对象
 */
public class TxtBuilder implements Builder {
	/**
	 * 用来记录构建的文件的内容，相当于产品
	 */
	private StringBuffer buffer = new StringBuffer();
	public void buildBody(
			Map<String, Collection<ExportDataModel>> mapData) {
		for(String tblName : mapData.keySet()){
			//先拼接表名称
			buffer.append(tblName+"\n");
			//然后循环拼接具体数据
			for(ExportDataModel edm : mapData.get(tblName)){
				buffer.append(edm.getProductId()+","+edm.getPrice()+","+edm.getAmount()+"\n");
			}
		}
	}
	public void buildFooter(ExportFooterModel efm) {
		buffer.append(efm.getExportUser());
	}
	public void buildHeader(ExportHeaderModel ehm) {
		buffer.append(ehm.getDepId()+","+ehm.getExportDate()+"\n");
	}	
	public StringBuffer getResult(){
		return buffer;
	}	
}

/**
 * 实现导出数据到XML文件的的构建器对象
 */
public class XmlBuilder implements Builder {
	/**
	 * 用来记录构建的文件的内容，相当于产品
	 */
	private StringBuffer buffer = new StringBuffer();
	public void buildBody(
			Map<String, Collection<ExportDataModel>> mapData) {
		buffer.append("  <Body>\n");
		for(String tblName : mapData.keySet()){
			//先拼接表名称
			buffer.append("    <Datas TableName=\""+tblName+"\">\n");
			//然后循环拼接具体数据
			for(ExportDataModel edm : mapData.get(tblName)){
				buffer.append("      <Data>\n");
				buffer.append("        <ProductId>"+edm.getProductId()+"</ProductId>\n");
				buffer.append("        <Price>"+edm.getPrice()+"</Price>\n");
				buffer.append("        <Amount>"+edm.getAmount()+"</Amount>\n");
				buffer.append("      </Data>\n");
			}
			buffer.append("    </Datas>\n");
		}
		buffer.append("  </Body>\n");
	}

	public void buildFooter(ExportFooterModel efm) {
		buffer.append("  <Footer>\n");
		buffer.append("    <ExportUser>"+efm.getExportUser()+"</ExportUser>\n");
		buffer.append("  </Footer>\n");
		buffer.append("</Report>\n");
	}

	public void buildHeader(ExportHeaderModel ehm) {
		buffer.append("<?xml version='1.0' encoding='gb2312'?>\n");
		buffer.append("<Report>\n");
		buffer.append("  <Header>\n");
		buffer.append("    <DepId>"+ehm.getDepId()+"</DepId>\n");
		buffer.append("    <ExportDate>"+ehm.getExportDate()+"</ExportDate>\n");
		buffer.append("  </Header>\n");
	}
	public StringBuffer getResult(){
		return buffer;
	}
	
}

/**
 * 指导者，指导使用构建器的接口来构建输出的文件的对象
 */
public class Director {
	/**
	 * 持有当前需要使用的构建器对象
	 */
	private Builder builder;
	/**
	 * 构造方法，传入构建器对象
	 * @param builder 构建器对象
	 */
	public Director(Builder builder) {
		this.builder = builder;
	}
	/**
	 * 指导构建器构建最终的输出的文件的对象
	 * @param ehm 文件头的内容
	 * @param mapData 数据的内容
	 * @param efm 文件尾的内容
	 */
	public void construct(ExportHeaderModel ehm,Map<String,Collection<ExportDataModel>> mapData,ExportFooterModel efm) {
		//1：先构建Header
		builder.buildHeader(ehm);
		//2：然后构建Body
		builder.buildBody(mapData);
		//3：然后构建Footer
		builder.buildFooter(efm);
	}
}

public class Client {
	public static void main(String[] args) {
		//准备测试数据
		ExportHeaderModel ehm = new ExportHeaderModel();
		ehm.setDepId("一分公司");
		ehm.setExportDate("2010-05-18");
		
		Map<String,Collection<ExportDataModel>> mapData = new HashMap<String,Collection<ExportDataModel>>();
		Collection<ExportDataModel> col = new ArrayList<ExportDataModel>();
		
		ExportDataModel edm1 = new ExportDataModel();
		edm1.setProductId("产品001号");
		edm1.setPrice(100);
		edm1.setAmount(80);
		
		ExportDataModel edm2 = new ExportDataModel();
		edm2.setProductId("产品002号");
		edm2.setPrice(99);
		edm2.setAmount(55);		
		//把数据组装起来
		col.add(edm1);
		col.add(edm2);		
		mapData.put("销售记录表", col);
		
		ExportFooterModel efm = new ExportFooterModel();
		efm.setExportUser("张三");
		
		//测试输出到文本文件
		TxtBuilder txtBuilder = new TxtBuilder();
		//创建指导者对象
		Director director = new Director(txtBuilder);
		director.construct(ehm, mapData, efm);
		//把要输出的内容输出到控制台看看
		System.out.println("输出到文本文件的内容：\n"+txtBuilder.getResult());
		//测试输出到xml文件
		XmlBuilder xmlBuilder = new XmlBuilder();
		Director director2 = new Director(xmlBuilder);
		director2.construct(ehm, mapData, efm);
		//把要输出的内容输出到控制台看看
		System.out.println("输出到XML文件的内容：\n"+xmlBuilder.getResult());
		
	}
}
```

> 请参考github designpattern项目下builder下example3包中的代码示例

## 认识生成器模式
### 生成器模式的功能
生成器模式的主要功能是构建复杂的产品，而且是细化的，分步骤的构建产品，也就是生成器模式重在解决一步一步构造复杂对象的问题。

更为重要的是，这个构建的过程是统一的，固定不变的，变化的部分放到生成器部分了，只要配置不同的生成器，那么同样的构建过程，就能构建出不同的产品表示来。

直白点说，生成器模式的重心在于分离构建算法和具体的构造实现，从而使得构建算法可以重用，具体的构造实现可以很方便的扩展和切换，从而可以灵活的组合来构造出不同的产品对象。

### 生成器模式的构成
生成器模式分成两个重要部分：
（1）Builder接口，是定义了如何构建各个部件，也就是知道每个部件功能如何实现，以及如何装配这些部件到产品中去；
（2）Director，Director是知道如何组合来构建产品，也就是说Director负责整体的构建算法，而且通常是分步骤的来执行。

在Director实现整体构建算法的时候，遇到需要创建和组合具体部件的时候，就会把这些功能通过委托，交给Builder去完成。
 
 
### 生成器模式的实现
#### 生成器的实现

> 实际上在Builder接口的实现中，每个部件构建的方法里面，除了部件装配外，也可以实现如何具体的创建各个部件对象，也就是说每个方法都可以有两部分功能，一个是创建部件对象，一个是组装部件。

> 在构建部件的方法里面可以实现选择并创建具体的部件对象，然后再把这个部件对象组装到产品对象中去，这样一来，Builder就可以和工厂方法配合使用了。

> 再进一步，如果在实现Builder的时候，只有创建对象的功能，而没有组装的功能，那么这个时候的Builder实现跟抽象工厂的实现是类似的。

> 这种情况下，Builder接口就类似于抽象工厂的接口，Builder的具体实现就类似于具体的工厂，而且Builder接口里面定义的创建各个部件的方法也是有关联的，这些方法是构建一个复杂对象所需要的部件对象

#### 指导者的实现

> 指导者承担的是整体构建算法部分，是相对不变的部分。因此在实现指导者的时候，把变化的部分分离出去是很重要的。

> 其实指导者分离出去的变化部分，就到了生成器那边，指导者知道整体的构建算法，就是不知道如何具体的创建和装配部件对象。

> 因此真正的指导者实现，并不仅仅是如同前面示例那样，简单的按照一定顺序调用生成器的方法来生成对象，并没有这么简单。应该是有较为复杂的算法和运算过程，在运算过程中根据需要，才会调用生成器的方法来生成部件对象。

#### 指导者和生成器的交互

在生成器模式里面，指导者和生成器的交互，是通过生成器的那些buildPart方法来完成的。指导者通常会实现比较复杂的算法或者是运算过程，在实际中很可能会有这样的情况：
a：在运行指导者的时候，会按照整体构建算法的步骤进行运算，可能先运行前几步运算，到了某一步骤，需要具体创建某个部件对象了，然后就调用Builder中创建相应部件的方法来创建具体的部件。同时，把前面运算得到的数据传递给Builder，因为在Builder内部实现创建和组装部件的时候，可能会需要这些数据
b：Builder创建完具体的部件对象后，会把创建好的部件对象返回给指导者，指导者继续后续的算法运算，可能会用到已经创建好的对象
c：如此反复下去，直到整个构建算法运行完成，那么最终的产品对象也就创建好了
通过上面的描述，可以看出指导者和生成器是需要交互的，方式就是通过生成器方法的参数和返回值，来回的传递数据。事实上，指导者是通过委托的方式来把功能交给生成器去完成。

#### 返回装配好的产品的方法

在标准的生成器模式里面，在Builder实现里面会提供一个返回装配好的产品的方法，在Builder接口上是没有的。它考虑的是最终的对象一定要通过部件构建和装配，才算真正创建了，而具体干活的就是这个Builder实现，虽然指导者也参与了，但是指导者是不负责具体的部件创建和组装的，因此客户端是从Builder实现里面获取最终装配好的产品。
 
#### 关于被构建的产品的接口

在使用生成器模式的时候，大多数情况下是不知道最终构建出来的产品是什么样的，所以在标准的生成器模式里面，一般是不需要对产品定义抽象接口的，因为最终构造的产品千差万别，给这些产品定义公共接口几乎是没有意义的。


#### 使用生成器模式构建复杂对象

考虑这样一个实际应用，要创建一个保险合同的对象，里面很多属性的值都有约束，要求创建出来的对象是满足这些约束规则的。约束规则比如：保险合同通常情况下可以和个人签订，也可以和某个公司签订，但是一份保险合同不能同时与个人和公司签订。这个对象里面有很多类似这样的约束，那么该如何来创建这个对象呢？
 
1：使用Builder模式来构建复杂对象，先不考虑带约束
 
```
/**
 * 保险合同的对象
 */
public class InsuranceContract {
	/**
	 * 保险合同编号
	 */
	private String contractId;
	/**
	 * 被保险人员的名称，同一份保险合同，要么跟人员签订，要么跟公司签订，
	 * 也就是说，"被保险人员"和"被保险公司"这两个属性，不可能同时有值
	 */
	private String personName;
	/**
	 * 被保险公司的名称
	 */
	private String companyName;
	/**
	 * 保险开始生效的日期
	 */
	private long beginDate;
	/**
	 * 保险失效的日期，一定会大于保险开始生效的日期
	 */
	private long endDate;
	/**
	 * 示例：其它数据
	 */
	private String otherData;
	
	/**
	 * 构造方法，访问级别是同包能访问
	 */
	InsuranceContract(ConcreteBuilder builder){
		this.contractId = builder.getContractId();
		this.personName = builder.getPersonName();
		this.companyName = builder.getCompanyName();
		this.beginDate = builder.getBeginDate();
		this.endDate = builder.getEndDate();
		this.otherData = builder.getOtherData();
	}
	/**
	 * 示意：保险合同的某些操作
	 */
	public void someOperation(){
		System.out.println("Now in Insurance Contract someOperation=="+this.contractId);
	}
}
	
/**
 * 构造保险合同对象的构建器
 */
public class ConcreteBuilder {
	private String contractId;
	private String personName;
	private String companyName;
	private long beginDate;
	private long endDate;
	private String otherData;
	/**
	 * 构造方法，传入必须要有的参数
	 * @param contractId 保险合同编号
	 * @param beginDate 保险开始生效的日期
	 * @param endDate 保险失效的日期
	 */
	public ConcreteBuilder(String contractId,long beginDate,long endDate){
		this.contractId = contractId;
		this.beginDate = beginDate;
		this.endDate = endDate;
	}
	/**
	 * 选填数据，被保险人员的名称
	 * @param personName  被保险人员的名称
	 * @return 构建器对象
	 */
	public ConcreteBuilder setPersonName(String personName){
		this.personName = personName;
		return this;
	}
	/**
	 *  选填数据，被保险公司的名称
	 * @param companyName 被保险公司的名称
	 * @return 构建器对象
	 */
	public ConcreteBuilder setCompanyName(String companyName){
		this.companyName = companyName;
		return this;
	}
	/**
	 * 选填数据，其它数据
	 * @param otherData 其它数据
	 * @return 构建器对象
	 */
	public ConcreteBuilder setOtherData(String otherData){
		this.otherData = otherData;
		return this;
	}
	/**
	 * 构建真正的对象并返回
	 * @return 构建的保险合同的对象
	 */
	public InsuranceContract build(){
		return new InsuranceContract(this);
	}

//省略getXXX方法

}
	

public class Client {
	public static void main(String[] args) {
		//创建构建器
		ConcreteBuilder builder = new ConcreteBuilder("001",12345L,67890L);
		//设置需要的数据，然后构建保险合同对象
		InsuranceContract contract = builder.setPersonName("张三").setOtherData("test").build();
		//操作保险合同对象的方法
		contract.someOperation();
	}
}	
```
 
 > 请参考github designpattern项目下builder下example4包中的代码示例

2：使用Builder模式来构建复杂对象，考虑带约束规则

```
/**
 * 构造保险合同对象的构建器
 */
public class ConcreteBuilder {
	
	//其他代码跟不带约束条件的一样
	
	/**
	 * 构建真正的对象并返回
	 * @return 构建的保险合同的对象
	 */
	public InsuranceContract build(){
		if(contractId==null || contractId.trim().length()==0){
			throw new IllegalArgumentException("合同编号不能为空");
		}
		boolean signPerson = personName!=null && personName.trim().length()>0;
		boolean signCompany = companyName!=null && companyName.trim().length()>0;
		if(signPerson && signCompany){
			throw new IllegalArgumentException("一份保险合同不能同时与人和公司签订");
		}		
		if(signPerson==false && signCompany==false){
			throw new IllegalArgumentException("一份保险合同不能没有签订对象");
		}
		if(beginDate<=0){
			throw new IllegalArgumentException("合同必须有保险开始生效的日期");
		}
		if(endDate<=0){
			throw new IllegalArgumentException("合同必须有保险失效的日期");
		}
		if(endDate<=beginDate){
			throw new IllegalArgumentException("保险失效的日期必须大于保险生效日期");
		}
		return new InsuranceContract(this);
	}
	

}
```

 > 请参考github designpattern项目下builder下example5包中的代码示例

3：进一步，把构建器对象和被构建对象合并

```
/**
 * 保险合同的对象
 */
public class InsuranceContract {
<!--外部类中没有属性的setXXX方法，只有getXXX-->
	/**
	 * 保险合同编号
	 */
	private String contractId;
	/**
	 * 被保险人员的名称，同一份保险合同，要么跟人员签订，要么跟公司签订，
	 * 也就是说，"被保险人员"和"被保险公司"这两个属性，不可能同时有值
	 */
	private String personName;
	/**
	 * 被保险公司的名称
	 */
	private String companyName;
	/**
	 * 保险开始生效的日期
	 */
	private long beginDate;
	/**
	 * 保险失效的日期，一定会大于保险开始生效的日期
	 */
	private long endDate;
	/**
	 * 示例：其它数据
	 */
	private String otherData;
	
	/**
	 * 构造方法，访问级别是私有的
	 */
	private InsuranceContract(ConcreteBuilder builder){
		this.contractId = builder.contractId;
		this.personName = builder.personName;
		this.companyName = builder.companyName;
		this.beginDate = builder.beginDate;
		this.endDate = builder.endDate;
		this.otherData = builder.otherData;
	}
	
	/**
	 * 构造保险合同对象的构建器
	 */
	public static class ConcreteBuilder {
		private String contractId;
		private String personName;
		private String companyName;
		private long beginDate;
		private long endDate;
		private String otherData;
		/**
		 * 构造方法，传入必须要有的参数
		 * @param contractId 保险合同编号
		 * @param beginDate 保险开始生效的日期
		 * @param endDate 保险失效的日期
		 */
		public ConcreteBuilder(String contractId,long beginDate,long endDate){
			this.contractId = contractId;
			this.beginDate = beginDate;
			this.endDate = endDate;
		}
		/**
		 * 选填数据，被保险人员的名称
		 * @param personName  被保险人员的名称
		 * @return 构建器对象
		 */
		public ConcreteBuilder setPersonName(String personName){
			this.personName = personName;
			return this;
		}
		/**
		 *  选填数据，被保险公司的名称
		 * @param companyName 被保险公司的名称
		 * @return 构建器对象
		 */
		public ConcreteBuilder setCompanyName(String companyName){
			this.companyName = companyName;
			return this;
		}
		/**
		 * 选填数据，其它数据
		 * @param otherData 其它数据
		 * @return 构建器对象
		 */
		public ConcreteBuilder setOtherData(String otherData){
			this.otherData = otherData;
			return this;
		}
		/**
		 * 构建真正的对象并返回
		 * @return 构建的保险合同的对象
		 */
		public InsuranceContract build(){
			if(contractId==null || contractId.trim().length()==0){
				throw new IllegalArgumentException("合同编号不能为空");
			}
			boolean signPerson = personName!=null && personName.trim().length()>0;
			boolean signCompany = companyName!=null && companyName.trim().length()>0;
			if(signPerson && signCompany){
				throw new IllegalArgumentException("一份保险合同不能同时与人和公司签订");
			}		
			if(signPerson==false && signCompany==false){
				throw new IllegalArgumentException("一份保险合同不能没有签订对象");
			}
			if(beginDate<=0){
				throw new IllegalArgumentException("合同必须有保险开始生效的日期");
			}
			if(endDate<=0){
				throw new IllegalArgumentException("合同必须有保险失效的日期");
			}
			if(endDate<=beginDate){
				throw new IllegalArgumentException("保险失效的日期必须大于保险生效日期");
			}
			
			return new InsuranceContract(this);
		}
	}
	
	/**
	 * 示意：保险合同的某些操作
	 */
	public void someOperation(){
		System.out.println("Now in Insurance Contract someOperation=="+this.contractId);
	}
}

public class Client {
	public static void main(String[] args) {
		//创建构建器
		InsuranceContract.ConcreteBuilder builder = new InsuranceContract.ConcreteBuilder("001",12345L,67890L);
		//设置需要的数据，然后构建保险合同对象
		InsuranceContract contract = builder.setPersonName("张三").setOtherData("test").build();
		//操作保险合同对象的方法
		contract.someOperation();
	}
}
```

 > 请参考github designpattern项目下builder下example6包中的代码示例


## 生成器模式的优缺点  

 1. 松散耦合 
 2. 可以很容易的改变产品的内部表示
 3. 更好的复用性

## 生成器模式的本质  

> 分离整体构建算法和部件构造

 
## 何时选用生成器模式  
1. 如果创建对象的算法，应该独立于该对象的组成部分以及它们的装配方式时
2. 如果同一个构建过程有着不同的表示时


  [1]: http://static.zybuluo.com/wangkaihua5/1l89ejjwpxehvzhc6zxja3km/image_1cf2nei5jou68e915eg13oouvk9.png
  [2]: http://static.zybuluo.com/wangkaihua5/d44p5hits7a380849gy1f1vr/image_1cf6rul0pvh91lsqaaf15a210169.png