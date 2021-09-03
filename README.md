# Deploying-Selenium-Python-as-AWS-Lambda-Function
Detailed description of process to deploy selenium python script as AWS lambda function to allow for serverless testing. Selenium runs off of a headless chrome binary and is driver by a chrome driver, each deployed as layers to the lambda function.

Summary:
  - Creating AWS Account
  - Packaging and Deploying Selenium Library and Chrome Driver as Layers
  - Creating a Lambda Function
  - Connecting Layers
  - Manual Testing
  - Scheduling Testing

Creating AWS Account:
  1.) Open the Amazon Web Services (AWS) home page.
  2.) Choose Create an AWS Account.
  3.) Enter your account information, and then choose Continue. Be sure that you enter your account information correctly, especially your email address. If you enter your email         address incorrectly, you can't access your account.
  4.) Choose Personal or Professional.
  5.) Enter your company or personal information.
  6.) Read and accept the AWS Customer Agreement.
  7.) Choose Create Account and Continue.
  
Packaging and Deploying Selenium Library and Chrome Driver as Layers:
  1.) Login to AWS account and navigate to the "Lambda" service tab.
  2.) Under the additional resources tab, choose "Layers".
  3.) Click on "Create Layer", give it a name, then choose the appropriate .zip file for upload.
        i.e. If you are creating the layer for selenium, name it "selenium" and choose the "selenium.zip" folder included in this repository.
  4.) Choose compatible runtimes for the contents of the .zip folder you just chose. This is optional but helps to find the right layer later.
  5.) Click "Create" and allow the layer to intialize.
  6.) Repeat this process for the chromedriver layer as well. This .zip folder contains both the headless chrome binary and the chromedriver.
  7.) After both layers are initialized, you should see both of them in the layers tab.
  
Creating Lambda Function:
  1.) Inside the Lambda dashboard, navigate to the "Functions" page using the panel on the left.
  2.) Choose "Create function" and "Author from Scratch".
  3.) Give the function an appropriate name and choose the runtime that correlates to the selenium library you uploaded to the layer.
        i.e. For the selenium.zip folder in this repository, Python 3.6 is the runtime needed.
  4.) Permissions and Advanced Settings can be left as is, then click "Create function".
  5.) This will initialize a blank lambda function which is ready to be filled in with the desired webscraping function.
  6.) If using Python, you can copy and paste your code directly from an IDE into the lambda function window to deploy it. This includes all needed import statements. To make full       use of the chromedriver, a few options are needed:
  
        from selenium import webdriver
        from selenium.webdriver.chrome.options import Options
        
        options = Options()
        options.add_argument('--headless')
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-gpu')
        options.add_argument('--window-size=1280x1696')
        options.add_argument('--hide-scrollbars')
        options.add_argument('--enable-logging')
        options.add_argument('--log-level=0')
        options.add_argument('--v=99')
        options.add_argument('--single-process')
        options.add_argument('--ignore-certificate-errors')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36')
        options.binary_location = r"/tmp/headless-chromium"
        browser = webdriver.Chrome("/tmp/chromedriver",chrome_options=options)
        
 Connecting Layers:
  1.) In order for the lambda function to make use of selenium and the chromedriver, the layers made earlier must be associated with the function.
  2.) To do this, navigate to the main page for the lambda function you wish to use, then click on the "Layers" section of the function overview.
  3.) Choose "Add a layer", then "Custom layers", then one of the two needed layers from the dropdown.
  4.) Repeat this for both the selenium and chromedriver layers, giving the function the capability to access the web.
  5.) One extra step is needed to get the layers working correctly with the function. Using the layers method to attatch libraries negates the need to package the function in a         container beforehand, making for easier editing on the fly. However, AWS stores layers in a folder names /opt, which happens to be a read only location. This means that the       chromedriver and binary cannot be executed from this location as they do not have the correct permissions. Luckily, a temporary folder called /tmp is generated each time a         test is ran which has all the permissions needed to execute the layers. All that's needed is to copy the contents of the /opt folder into the /tmp folder at the beginning of       each test. Here is the code needed to do this, assuming the .zip containing the chromedriver and binary is named "chromedriver":
  
      import os
      
      os.system("cp -r /opt/chromedriver/chromedriver /tmp/chromedriver")
      os.system("cp -r /opt/chromedriver/headless-chromium /tmp/headless-chromium")
      os.chmod("/tmp/chromedriver", 0o777)
      os.chmod("/tmp/headless-chromium", 0o777)

Manual Testing:
  1.) Assuming all of the previous steps were carried out correctly, you should be ready to run your first lambda function.
  2.) Here is a sample lambda function which uses the included layers to open google.com, then prints a message saying "Success" if the function was run completely:
    
    from selenium import webdriver
    from selenium.webdriver.chrome.options import Options
    import os
 
    def lambda_handler(event, context):
      #Copy contents of /opt to /tmp for execution:
      os.system("cp -r /opt/chromedriver/chromedriver /tmp/chromedriver")
      os.system("cp -r /opt/chromedriver/headless-chromium /tmp/headless-chromium")
      os.chmod("/tmp/chromedriver", 0o777)
      os.chmod("/tmp/headless-chromium", 0o777)
    
      #Initliaze browser:  
      options = Options()
      options.add_argument('--headless')
      options.add_argument('--no-sandbox')
      options.add_argument('--disable-gpu')
      options.add_argument('--window-size=1280x1696')
      options.add_argument('--hide-scrollbars')
      options.add_argument('--enable-logging')
      options.add_argument('--log-level=0')
      options.add_argument('--v=99')
      options.add_argument('--single-process')
      options.add_argument('--ignore-certificate-errors')
      options.add_argument('--disable-dev-shm-usage')
      options.add_argument('user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36')
      options.binary_location = r"/tmp/headless-chromium"

      #Initialize browser and open google.com:
      browser = webdriver.Chrome("/tmp/chromedriver",chrome_options=options)
      browser.get('https://www.google.com')
      
      #Close browser and print message:
      browser.quit()
      print("Success")
      
  3.) To run a manual test, click the orange "Test" button, give the test a name, and configure the test with the Hello World settings. Do not change the test inputs from default. 
  4.) Deploy the lambda function, then click the test button. If the function worked as intended, the response should be "null" and the "Success" message should be present in the       function logs. 
  5.) If this all went smoothly, feel free to edit the lambda function to your liking.
  
Scheduling Testing:
  1.) Manually testing is great, but the real use case for this approach is automated scheduled testing. For example, let's say you want to have a bot do some webscraping on a           particular website every Monday at 2:00PM UTC. This can be done easily using CloudWatch Events.
  2.) To begin, find the "Add trigger" button in the function overview section of your lambda function.
  3.) When asked for trigger configuration, choose "EventBridge (CloudWatch Events)".
  4.) Now choose "Create a new rule", give it a name and description, and select "Schedule expression".
  5.) A Cron expression is now needed to determine the exact specifications to run your function. Information on Cron expressions can be found here:                  https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html
      i.e. For the Monday at 2:00PM UTC example, the Cron expression would be: (0 14 ? * MON * ) Remember Cron expressions are in UTC, so a time zone conversion may be needed. 
  6.) Once all information is input, click "Add" to create the rule. From then on, your funciton will be ran at the scheduled time. 
  7.) Detailed logs on the schduled rule can be found by clicking on the rule, then finding the "Metrics for the rule" button.
  8.) Select "Log groups" under the "Logs" tab on the left, and select your rule's name. 
  
This concludes the Selenium Python to AWS Lambda tutorial. Happy testing!
