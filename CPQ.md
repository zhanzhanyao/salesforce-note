# Sales Cloud
Salesforce提供的CRM系统的一部分，专为销售人员创建。  
Sales Cloud标准对象：
- 销售线索leads，
- 客户account，
- 联系人contact，
- 商机opportinuty


# CPQ
salesforce CPQ的报价过程： 选择产品-计算价格-生成PDF报价单-管理合同和复购
- Configure: 客户想要购买什么产品
- Price: 单价及总价  
- Quote：一个PDF并附带所有报价的细节，给客户提供购买信息  

![quote](image/quote.png)

## CPQ API

### Quote
#### QuoteModel

    public class QuoteModel{
        public SBQQ_Quote_c record;
        public QuoteLineModel [] lineItems;
        public QuoteLineGroupModel [] lineItemGroups;
        public Integer nextKey;
        public Boolean applyAdditionalDiscountLast;
        public Boolean applyPartmerDiscountFirst;
        public Boolean channelDiscountOffList;
        public Decimal customerTotal;
        public Decimal netTotal;
        public Decimal netNonSegmentTotal;
    }

#### Quote API

##### 1. Save Quote 
    public with sharing class QuoteSaver{
        public QuoteModel save(QuoteModel quote){
            String quoteJSON = SBQQ.ServiceRouter.save('SBQQ.QuoteAPI.QUOTESaver', JSON.serialize(quote));
            return (QuoteModel) JSON.deserialize(quoteJSON, QuoteModel.class);
        }
    }
    
    QuoteSaver saver = new QuoteSaver();
    QuoteModel saveQuote = saver.save(quoteModel);
    System.debug(saveQuote);

##### 2. Read Quote
Reads a quote from a CPQ quote ID

    public with sharing class QuoteReader{
        public QuoteModel read(String quoteId){
            String quoteJSON = SBQQ.ServiceRouter.read('SBQQ.QuoteAPI.QuoteReader', quoteId);
            return (QuoteModel) JSON.deserialize(quoteJSON, QuoteModel.class)
        }
    }
    
    QuoteReader reader = new QuoteReader();
    QuoteModel quote = reader.read('a0Wf100000J1vk1');
    System.debug(quote);

#### 3. Calculate
When you execute the Calculate API with an Apex trigger, you also need to create a quote calculator callback class.   

    global with sharing class MyCallback implements SBQQ.CalculateCallback{
        global void callback(String quoteJSON){
            SBQQ.ServiceRouter.save('SBQQ.QuoteAPI.QuoteSaver', quoteJson);
        }
    }

    public with sharing class QuoteCalculator{
        public void calculate(QuoteModel quote, String callbackClass){
            QuoteCalculatorContext ctx = new QuoteCalculatorContext(quote, callbackClass);
            SBQQ.ServiceRouter.load('SBQQ.QuoteAPI.QuoteCalculator', null, JSON.serialize(ctx));
        }
        private class QuoteCalculatorContext{
            private QuoteModel quote;
            private String callbackClass;
            
            private QuoteCalculatorContext(QuoteModel quote, String callbackClass){
                this.quote = quote;
                this.callbackClass = callbackClass;
            }
        }
    }

    QuoteModel quotemodel;
    quoteModel.lineItems[0].record.SBQQ_quantity_c = 2;
    QuoteCalculator calculator = new QuoteCalculator();
    calculator.calculate(quoteModel, 'MyCallback)

#### 4. Validate Quote

    public with sharing class Validator{
        public List<String> validate(QuoteModel quote){
            String res=SBQQ.ServiceRouter.load('SBQQ.QuoteAPI.QuoteValidator',null, JSON.serialize(quote));
            return (List<String>) JSON.deserialize(res, List<String>.class);
        }
    }
    
    QuoteModel quoteModel;
    Validator validator = new Validator();
    List<String> msgs = validator.validate(quoteModel);
    System.debug(msgs)

#### 5. Add Products
Receive a CPQ quote, product collection, and quote group key in a request, 
and return a Quote model with all provided products added as qupte lines.

    public with sharing class ProductAdder{
        public QuoteModel add(QuoteModel quote, ProductModel[] products, Integer groupKey){
            AddProductsContext ctx = new AddProductsContext(quote, products, groupKey);
            String quoteJSON = SBQQ.ServiceRouter.load('SBQQ.QuoteAPI.QuoteProductAdder',null,JSON.serialize(cxt));
            return (QuoteModel) JSON.deserialize(quoteJSON, QuoteModel.class);
        }
    
        private class AddProductsContext{
            private QuoteModel quote;
            private ProductModel products;
            private Integer groupKey;
            private final Boolean ignoreCalculate = true;
    
            private AddProductsContext(QuoteModel quote, ProductModel[] products, Integer groupKey){
                this.quote = quote;
                this.products = products;
                this.groupKey = groupKey;
            }
        }
    }

    QuoteModel quoteModel;
    productModel;
    
    LIst<ProductModel> productModels = new LIst<ProductModel>();
    productModels.add(productModel);
    ProductAdder adder = new ProductAdder();
    QuoteModel quoteWithProducts = adder.add(quoteModel, productModels, 0);
    System.debug(quoteWithProducts);


#### 6. Read Product
Takes the request's product ID, pricebook ID, and currency code and returns a Product model.

    public with sharing class ProductReader{

        public ProductModel read(Id productId, Id pricebookId, String currencyCode){
            ProductReaderContext ctx = new ProductReaderContext(pricebookId, currencyCode);
            String productJSON = SBQQ.ServiceRouter.load('SBQQ.ProductLoader', productId, JSON.serialize(ctx));
            return (ProductModel) JSON.deserialize(productJSON, ProductModel.class);
        }
    
        private class ProductReaderContext{
            private Id pricebookId;
            private String currencyCode;
    
            private ProductReaderContext(Id pricebookId, String currencyCode){
                this.pricebookId = pricebookId;
                this.currencyCode = currencyCode;
            }
        }
    }

    ProductReader reader = new ProductReader();
    ProductModel product = reader.read('01tj0000003P1SN', '01sj0000003THhKAAW', 'USD');
    System.debug(product);

#### 7. Create and Save Quote Proposal

    public with sharing class GenerateQuoteproposal{
        public String save(QuoteProposalModel context){
            return SBQQ.ServiceRouter.save('SBQQ.QuoteDocumentAPI.Save', JSON.serialize(context));
        }
    }
    
    QuoteProposalModel model = new Quote QuoteProposalModel();
    model.quoteId = 'a0n0R000000jhVC';
    model.templateId = 'a0l0R000000vahe';
    GenerateQuoteproposal proposalGenerator = new GenerateQuoteproposal();
    String jobId = proposalGenerator.save(model)d
    System.debug(jobId);

#### 8. Quote Term Reader

    public with sharing class QuotoTermReader{
        public List<QuoteTermModel> load(Id quoteId, Id templateId, String language){
            TermContext ctx = new TermContext(templateId, language);
            String quoteTermsJSON = SBQQ.ServiceRouter.load(SBQQ.QuoteTermAPI.Load', quoteId, JSON.serialize(ctx));
            return (List<QuoteTermModel>) JSON.deserialize(quoteTermsJSON, List<QuoteTermModel>.class);
    
        }
    
        private class TermContext{
            private Id templateId;
            private String language;
    
            private TermContext(Id templateId, String language){
                this.templateId = templateId;
                this.language = language;
            }
        }
    }

    QuotoTermReader quotoTermReader = new QuotoTermReader();
    List<QuoteTermModel> quoteTerms = quoteTermReader.load('a0x5C000000G1CV', 'a0v5C000000jTgr', 'es');
    System.debug(quoteTerms)


### Contract
#### 1. Contract Amender API
Receive a CPQ contract ID in a request, and return quote data for am amendment quote.

    // the amendment context is not used
    public with sharing class ContractAmender{
        public QuoteModel load(String contractId){
            String quoteJSON = SBQQ.ServiceRouter.load('SBQQ.ContractManipulationAPI.ContractAmender', contractId, null);
            return (QuoteModel) JSON.deserialize(quoteJSON, QuoteModel.class);
        }
    }
    
    ContractAmender amender = new ContractAmender();
    QuoteModel quote = amender.load('8001D000000AUlg'); // example id
    System.debug(quote);


    // the amendment context is used
    public with sharing class ContractAmenderTest {
        public QuoteModel load(String contractId, String context) {
            String quoteJSON = SBQQ.ServiceRouter.load('SBQQ.ContractManipulationAPI.ContractAmender', contractId, context);
            return (QuoteModel) JSON.deserialize(quoteJSON, QuoteModel.class);
    
    private with sharing class AmendmentContextTest {
        public Boolean returnOnlyQuoteId;
    }
    
    // Create an amendment context
    AmendmentContextTest context = new AmendmentContextTest();
    context.returnOnlyQuoteId = true; 
    
    // Invoke the ContractAmender API
    String contextJson = JSON.serialize(context);
    ContractAmenderTest amender = new ContractAmenderTest();
    QuoteModel quote = amender.load('CONTRACT_ID', contextJson);

#### 2. Contract Renewer API
Receive a CPQ contract in a request, and return quote information for one or more renewal quotes.

    // Define a class to hold information about the contracts to renew
    private with sharing class CreateRenewalContext {
        public Id masterContractId;
        public Contract[] renewedContracts;
        public Boolean returnOnlyQuoteId;
    }
    
    //define a class to hold the serialized context and the returned quote information
    public with sharing class ContractRenewer {
        public QuoteModel[] load(String masterContractId, String serializedContext) {
            String quotesJSON = SBQQ.ServiceRouter.load('SBQQ.ContractManipulationAPI.ContractRenewer', masterContractId, serializedContext);
            return (QuoteModel[]) JSON.deserialize(quotesJSON, List<QuoteModel>.class);
        }
    }
    
    // populate the renewal context
    CreateRenewalContext context = new CreateRenewalContext();
    context.masterContractID = '4567';
    context.renewedContracts = [SELECT Id FROM Contract WHERE Id IN ('4567','8910')];
    // Set returnOnlyQuoteId to true if you only want the Quote ID and not the entire Quote model.
    context.returnOnlyQuoteId = true; 
    
    //Serialized the renewal context
    context = '{"masterContractId": "8001D000000AUlg", "renewedContracts":[{"attributes":{"type":"Contract"},"Id":"8001D000000AUlg"}, {"attributes":{"type":"Contract"},"Id":"8001D000000AVGt"}]}';
    String jsonContext = JSON.serialize(context);
    
    // renew the two contracts
    ContractRenewer renewer = new ContractRenewer();
    QuoteModel[] quotes = renewer.load(null, jsonContext);

### Generate Quote Document API

    public with sharing class GenerateQuoteProposal {
        public String save(QuoteProposalModel context) {
            return SBQQ.ServiceRouter.save('SBQQ.QuoteDocumentAPI.Save', JSON.serialize(context));
        }
    }
    
    QuoteProposalModel model = new QuoteProposalModel();
    model.quoteId = 'a0n0R000000jhVC';
    model.templateId = 'a0l0R000000vahe';
    GenerateQuoteProposal proposalGenerator = new GenerateQuoteProposal();
    String jobId = proposalGenerator.save(model);
    System.debug(jobId);

### Demo

    //the Id of the quote
    String quoteId = 'a0Wf100000J1vk1';
    
    //the Id of the product to add to the quote
    String productId = '01tj0000003P1SN';
    
    //the Id of the pricebook for the quote and product being added
    String pricebookId = '01sj0000003THhKAAW';
    
    //the currency code
    String currencyCode = 'USD';
    
    String quoteModel = SBQQ.ServiceRouter.read('SBQQ.QuoteAPI.QuoteReader', quoteId);
    
    String productModel = SBQQ.ServiceRouter.load('SBQQ.ProductAPI.ProductLoader', productId, '{"pricebookId" : "' + pricebookId + '", "currencyCode" : "' + currencyCode + '"}');
    
    String updatedQuoteModel = SBQQ.ServiceRouter.load('SBQQ.QuoteAPI.QuoteProductAdder', null, '{"quote" : ' + quoteModel + ', "products" : [' + productModel + '], "ignoreCalculate" : true}');
    
    String savedQuoteModel = SBQQ.ServiceRouter.save('SBQQ.QuoteAPI.QuoteSaver', updatedQuoteModel);

### Disabled Trigger

    SBQQ.TriggerControl.disable();
    try {
        // Do something simple and interesting without
        // running triggers.
        quote.MyStatus__c = ‘Red’;
        update quote;
    } finally {
    SBQQ.TriggerControl.enable();
    }

    if (SBQQ.TriggerControl.isEnabled()) {
        // Only run our logic if CPQ trigger logic is also enabled.
        myRelatedObject.Quote__c = quote.Id;
        insert myRelatedObject;
    }