# UT模板

```java
/*********************************************************
 * 文件名称：ServiceTest.java
 * 系统名称：交易银行系统V2.0
 * 模块名称：com.hundsun.tbsp.test
 * 功能说明：某某功能的test
 * 开发人员：wucp26649
 * 开发时间：2019/7/31 11:35
 * 修改记录：程序版本    修改日期    修改人员    修改单号    修改说明
 *********************************************************/
@RunWith(Parameterized.class)
@SpringBootTest(classes = ScfbizStarter.class)
public class ServiceTest extends TcBaseTest {
    private Request req;
    private static int no = 0;
    
    @Autowired
    private Service service;

    public ServiceTest(Request req) {
        this.req = req;
        this.initTbspRequest(req, "trCode", 10);
        no++;
    }

    @Parameterized.Parameters
    public static Collection<Object> data() {
        List<Request> list = CSVSupport.getListFromCSV(Request.class,
                "Request.csv" + "@3");
        Object[] obj = new Object[list.size()];
        for (int i = 0; i < list.size(); i++) {
            obj[i] = new Object[]{list.get(i)};
        }
        return Arrays.asList(obj);
    }

    @Test
    public void test() {
		List<String> delSqls = new ArrayList<>();
        try {
            if (no == 1) {
                mock();
                delSqls.addAll(DataAssist
                        .readDataFromCsv("",
                                ""));
                
                Response resp = service.get(req);
                
                Assert.assertEquals("S", resp.getRespType());
                
                checkCSV("Response.csv", resp.getScfProjHis(), 1);
                listMultCheck(resp.getScfProjHisList(), "Response.csv", "1");
                DBCheck("scfProj.csv", "SCF_PROJ", 1);

            }
        } finally {
            DataAssist.execSqls(delSqls, "");
            DBDeleteSingle("TBSP_JNL",
                    "TR_CODE", req.getHeadTrCode(),
                    "TENANT_ID", req.getHeadTenantId());
        }
    }
    
    @Mocked
    private CustUserService custUserService;
    @Mocked
    private CustomerService customerService;
    @Mocked
    private ScfAuthCheckComponent scfAuthCheckComponent;

    private void mock() {
        MockUp<CustomerService> customerServiceMockUp = new MockUp<CustomerService>() {
            @Mock
            CustOrganDTO getCustOrgan(String tenantId, String orgNo) {
                CustOrganDTO dto = new CustOrganDTO();
                //当前机构o
                if (orgNo.equals("90000")) {
                    dto.setOrgIsn("001");
                    dto.setOrgNo("90000");
                    dto.setOrgName("自测机构1");
                }
                //目标机构
                if (orgNo.equals("90001")) {
                    dto.setOrgIsn("001001");
                    dto.setOrgNo("90001");
                    dto.setOrgName("自测机构2");
                }
                return dto;
            }
        };
        customerService = customerServiceMockUp.getMockInstance();
        scfAuthCheckComponent = SpringUtils.getBean(ScfAuthCheckComponent.class);
        ReflectionTestUtils.setField(scfAuthCheckComponent, "customerService", customerService);

        MockUp<CustUserService> custUserServiceMockUp = new MockUp<CustUserService>() {
            @Mock
            CustUserDTO getCustUser(String tenantId, String userNo) {
                CustUserDTO dto = new CustUserDTO();
                dto.setUserType("2");
                dto.setUserNo("26649");
                return dto;
            }
        };

        custUserService = custUserServiceMockUp.getMockInstance();
        PageScfProjHisServiceImpl pageScfProjHisService = SpringUtils.getBean(PageScfProjHisServiceImpl.class);
        ReflectionTestUtils.setField(pageScfProjHisService, "custUserService", custUserService);
    }

}
```

