# Ebay-website-automation
Automates eBay search for "outdoor toys" using advanced filters: category, condition, returns, and location. Scrapes product names and URLs from results, verifies keyword presence, and exports valid entries to Excel. Fully handles browser control and dynamic content.
package selenium;

import com.aventstack.extentreports.*;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.Select;
import org.testng.annotations.*;

import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class Ebay extends BaseClass {
    public static ExtentReports extent;
    public static ExtentTest test;

    @BeforeSuite
    public void setupReport() {
        ExtentSparkReporter spark = new ExtentSparkReporter("EbayTestReport.html");
        spark.config().setTheme(Theme.DARK);
        spark.config().setDocumentTitle("Ebay Automation Report");
        spark.config().setReportName("Ebay Search Test");

        extent = new ExtentReports();
        extent.attachReporter(spark);
        extent.setSystemInfo("Tester", "Rajagomathi S");
        extent.setSystemInfo("OS", System.getProperty("os.name"));
        extent.setSystemInfo("Java Version", System.getProperty("java.version"));
    }

    @Test
    public void searchAndExtractEbayToys() {
        test = extent.createTest("Ebay Search for Outdoor Toys");

        try {
            // Launch browser
            String browser = "chrome";
            launchBrowser(browser);
            String screenshotPath1 = ScreenshotUtil.takeScreenshot(driver, "Step 1: Browser Launched");
            test.log(Status.PASS, "Browser launched successfully: " + browser, MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath1).build());

            // Navigate to Ebay Home page
            driver.get("https://www.ebay.com/");
            String screenshotPath2 = ScreenshotUtil.takeScreenshot(driver, "Step 2: Ebay Home Page");
            test.log(Status.PASS, "Navigated to eBay homepage.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath2).build());

            // Click on Advanced Search
            driver.findElement(By.linkText("Advanced")).click();
            String screenshotPath3 = ScreenshotUtil.takeScreenshot(driver, "Step 3: Advanced Search Clicked");
            test.log(Status.PASS, "Clicked on 'Advanced Search' link.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath3).build());

            // Enter keyword
            driver.findElement(By.id("_nkw")).sendKeys("Outdoor Toys");
            String screenshotPath4 = ScreenshotUtil.takeScreenshot(driver, "Step 4: Keyword Entered");
            test.log(Status.PASS, "Entered keyword 'Outdoor Toys'.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath4).build());
            BaseClass.waitInSeconds(2);

            // Select keyword match type
            new Select(driver.findElement(By.name("_in_kw"))).selectByVisibleText("Any words, any order");
            BaseClass.waitInSeconds(2);
            String screenshotPath5 = ScreenshotUtil.takeScreenshot(driver, "Step 5: Keyword Match Selected");
            test.log(Status.PASS, "Selected keyword match type.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath5).build());
            BaseClass.waitInSeconds(1);

            // Select category
            new Select(driver.findElement(By.id("s0-1-19-4[0]-7[3]-_sacat"))).selectByVisibleText("Toys & Hobbies");
            String screenshotPath6 = ScreenshotUtil.takeScreenshot(driver, "Step 6: Category Selected");
            test.log(Status.PASS, "Selected category.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath6).build());

            // Check "Title and description"
            driver.findElement(By.cssSelector("input.checkbox__control")).click();
            BaseClass.waitInSeconds(1);
            String screenshotPath7 = ScreenshotUtil.takeScreenshot(driver, "Step 7: Title and Description Checked");
            test.log(Status.PASS, "Checked 'Title and description'.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath7).build());

            // Check "New" condition
            driver.findElement(By.cssSelector("input.radio__control")).click();
            BaseClass.waitInSeconds(1);
            String screenshotPath8 = ScreenshotUtil.takeScreenshot(driver, "Step 8: New Condition Checked");
            test.log(Status.PASS, "Checked 'New' condition.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath8).build());

            // Check "Free returns" and "Returns accepted"
            driver.findElement(By.name("LH_FR")).click();
            driver.findElement(By.name("LH_RPA")).click();
            BaseClass.waitInSeconds(1);
            String screenshotPath9 = ScreenshotUtil.takeScreenshot(driver, "Step 9: Return Options Checked");
            test.log(Status.PASS, "Checked return options.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath9).build());

            // Select "Worldwide" location
            driver.findElements(By.cssSelector("input.radio__control")).get(10).click();
            BaseClass.waitInSeconds(1);
            String screenshotPath10 = ScreenshotUtil.takeScreenshot(driver, "Step 10: Worldwide Location Selected");
            test.log(Status.PASS, "Selected 'Worldwide' location.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath10).build());

            // Click Search
            driver.findElements(By.xpath("//*[@class='btn btn--primary']")).get(1).click();
            BaseClass.waitInSeconds(1);
            String screenshotPath11 = ScreenshotUtil.takeScreenshot(driver, "Step 11: Search Clicked");
            test.log(Status.PASS, "Clicked the Search button.", MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath11).build());

            // Extract item names and links
            List<WebElement> items = driver.findElements(By.tagName("a"));
            List<String[]> results = new ArrayList<>();
            test.log(Status.INFO, "Extracting item names and links...");

            for (WebElement item : items) {
                try {
                    String name = item.getText();
                    String link = item.getAttribute("href");

                    if (name.contains("Toys")) {
                         results.add(new String[]{name, link});
                         test.log(Status.INFO, "Found item: " + name + " -> " + link);
                    }
                } catch (StaleElementReferenceException e) {
                    test.log(Status.WARNING, "StaleElementReferenceException caught.");
                }
            }

            // Write to Excel
            String fileName = "OutdoorToys.xlsx";
            if (results.isEmpty()) {
                test.log(Status.WARNING, "No matching items found.");
            } else {
                writeToExcel(results, fileName);
                test.log(Status.PASS, "Excel file created: " + fileName);
            }

            test.log(Status.PASS, "Test completed successfully.");

        } catch (Exception e) {
            String screenshotPath = ScreenshotUtil.takeScreenshot(driver, "Error");
            test.log(Status.FAIL, "Error during test: " + e.getMessage(), MediaEntityBuilder.createScreenCaptureFromPath(screenshotPath).build());
        }
    }

    @AfterMethod
    public void tearDownMethod() {
        if (driver != null) {
            driver.quit();
        }
    }

    @AfterSuite
    public void tearDownReport() {
        if (extent != null) {
            extent.flush();
        }
    }
    // Method to write results to Excel
    public void writeToExcel(List<String[]> data, String fileName) {
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Outdoor Toys");
        Row header = sheet.createRow(0);
        header.createCell(0).setCellValue("Item Name");
        header.createCell(1).setCellValue("Item Link");

        int rowCount = 1;
        for (String[] rowData : data) {
            Row row = sheet.createRow(rowCount++);
            for (int i = 0; i < rowData.length; i++) {
                row.createCell(i).setCellValue(rowData[i]);
                
            }
        }
        // Auto-size columns
        sheet.autoSizeColumn(0);
        sheet.autoSizeColumn(1);

        try (FileOutputStream outputStream = new FileOutputStream(fileName)) {
            workbook.write(outputStream);
            workbook.close();
            System.out.println("Sucess : Excel printed sucessfully "+ fileName) ;
            } catch (IOException e) {
            e.printStackTrace();
            test.log(Status.FAIL, "Failed to write to Excel: " + e.getMessage());
        }
    }
}
