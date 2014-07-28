!SLIDE 
# Goals of a UI Test Framework #

!SLIDE bullets incremental smaller
# Definition of a Test Framework
* manages webdriver connections, whether remote or local
* manages configuration per env -- usernames, passwords, urls, etc
* integrates with jenkins
* provides a structure for your test code in the form of page objects
* provides a pleasant and easy environment to write tests

!SLIDE bullets incremental
# webdriver details are managed outside your test
* running on sauce or on your local machine
* which browser, os, etc (desired capabilities)
* connection to webdriver before your test.  
* disconnect from webdriver after your test.

!SLIDE bullets incremental
# Webdriver setup details are outside your test


!SLIDE bullets incremental smaller
# Do not do this:
    @@@ Python 
      def setUp(self):
        desired_capabilities = webdriver.DesiredCapabilities.FIREFOX
        desired_capabilities['version'] = ''
        desired_capabilities['platform'] = 'Windows 8'
        desired_capabilities['name'] = 'The Test Name'


!SLIDE bullets incremental smaller left
# Instead, do this:
* `ant -Dbrowser=firefox -Dtest=newfunnels/citizen_test.py`
* test code no longer specifies any of these details
* webdriver connects and disconnects before / after the suite automatically.


!SLIDE bullets incremental smaller
# Configuration is specified outside your test 
* the framework knows that you are in UAT or QAE or Dev or Prod
* Provides you correct users, passwords, urls for that environment

!SLIDE bullets incremental smaller
# Do not do this:
    @@@ Python 
      def someTest(self):
        self.driver.get('http://wsdc.uat.sungevity.com/citizensenergy')

!SLIDE bullets incremental smaller left
# Instead, do this:
    @@@ Python 
    def setUp(self):
      self.home_url = self.config['endpoints']['funnels']['citizensenergy']

    def someTest(self):
      home_page = CitizenHome(driver=self.driver, url=self.home_url)

      # Just showing that you have access to what env you are running in.
      # In this example, config['env'] is perhaps 'uat'
      print "My environment is: " + config['env']  

* Note, this uses a PageObject.  
* More on that to come.


!SLIDE bullets incremental smaller 
# Jenkins integration is via standard ant tasks
# Currently supported tests are:
* `ant start_sauce_tunnel`
* `ant test -Dtest=myproject/sometest.py -Denv=uat`
* convenience tasks so you don't have to pass in the environment...
* `ant testuat -Dtest=myproject/sometest.py`
* `ant testqae -Dtest=myproject/sometest.py`
* `ant testdev -Dtest=myproject/sometest.py`
* `ant testprod -Dtest=myproject/sometest.py`


!SLIDE bullets incremental smaller 
# Frameworks support definition and loading of PageObjects


!SLIDE bullets incremental smaller 
# Example of a PageObject

    @@@ Python 
    class MiniFunnel(Page):
      funnel_frame_id = 'funnelframe'
      minifunnel_id = 'minifunnel'
      postal_box_id = 'j_id0:funnelGeneratorForm:postalCodeId'

      def __init__(self, *args, **kwargs):
          super(MiniFunnel, self).__init__(*args, **kwargs)
          self.driver.switch_to_frame(self.funnel_frame_id )
          self.driver.switch_to_frame(self.minifunnel_id)
          self._define_elements()
  
      def _define_elements(self):
          self.postal_box = elem(self.driver,Locators.CSS_SELECTOR, self.postal_box_id)


      def set_postal_code(self, postal_code):
          self.postal_box.send_keys(postal_code + '\n')
          return ApexFunnel(driver=self.driver)


!SLIDE bullets incremental smaller 
# Points to Note Regarding That Page Object
* It's a way to represent a single page of the application 
* Is defined outside of your test but used within.
* It's reusable.  Once written, page objects are reused across any tests that need them.
* It contains locators and is the only place those should be updated.  Locators should not be found in tests.
* Should provide 'high level' actions that a business person could understand, like 'set_postal_code'

!SLIDE bullets incremental smaller 
# Using a PageObject

    @@@ Python
    def testFunnel(self):
        home_page = CitizenHome(driver=self.driver, url=self.home_url)
        minifunnel = home_page.select_minifunnel()

        apex_funnel = minifunnel.set_postal_code(self.person.zipcode)

        apex_funnel.fill_person_form(self.person)

!SLIDE bullets incremental smaller 
# Frameworks provide a pleasant and easy way to write tests

* Drop into debugger if test encounters an error
* Interactive execution and debug environment 
* Step through a test one line at a time.
