# Number Sequence in D365F&O
This is sought of an article plus a code repo for the implementation of Number Sequence in D365F&O

### Implementing it is a five step procedure: 
#### Step 1:
  - The fields selected for implementation should be a string and should have an EDT(Extended Data Type).
  - PS: Also set the relevant properties such as Label, Name, etc. 
#### Step 2: 
  - Creating a class that extends NumberSeqApplicationModule
  -  The code for this can be found in the the file (VendAccountNum.xpp)
  -  Copy the code and make the changes to suit your requirements
    
      ```
              public class VendAccountNum extends NumberSeqApplicationModule
                {
                    protected void loadModule()
                    {
                        // Create a new NumberSeqDatatype object
                        NumberSeqDatatype datatype = NumberSeqDatatype::construct();
                
                        // Set the properties of the datatype object
                        // Set the extended data type ID
                        datatype.parmDatatypeId(extendedTypeNum(VendNumEDT)); 
                        // Set the reference help text
                        datatype.parmReferenceHelp(literalStr("Unique key used for the cars.")); 
                        // Set whether the number sequence is continuous
                        datatype.parmWizardIsContinuous(false);
                        // Set whether manual input is allowed 
                        datatype.parmWizardIsManual(NoYes::No); 
                         // Set the number of pre-generated numbers to fetch ahead
                        datatype.parmWizardFetchAheadQty(10);
                        // Set whether changing down is allowed
                        datatype.parmWizardIsChangeDownAllowed(NoYes::No); 
                        // Set whether changing up is allowed
                        datatype.parmWizardIsChangeUpAllowed(NoYes::No); 
                        // Set the sort field for the number sequence
                        datatype.parmSortField(1); 
                
                        // Add a parameter type to the datatype object (e.g., DataArea)
                        datatype.addParameterType(NumberSeqParameterType::DataArea, true, false);
                
                        // Create the number sequence using the configured datatype
                        this.create(datatype);
                    }
                
                    // Return the number sequence module for this class (e.g., "Cust", "Vend", "Sales", "Purch")
                    public NumberSeqModule numberSeqModule()
                    {
                        return NumberSeqModule::Vend;
                    }
                
                    // Subscribe to the NumberSeqGlobal class and add the custom module to the module names map
                    [SubscribesTo(classstr(NumberSeqGlobal), delegatestr(NumberSeqGlobal, buildModulesMapDelegate))]
                    static void buildModulesMapSubsciber(Map numberSeqModuleNamesMap)
                    {
                        NumberSeqGlobal::addModuleToMap(classnum(VendAccountNum), numberSeqModuleNamesMap);
                    }
                }
        
#### Step 3: 
  - Creating a runnable class (job) to have the system process the class in step 2
  -  The code for this can be found in the the file (VendAccountNumRunnableClass.xpp)
  -  PS: Also Set the class as a Start Up Object
  -  The code is actually:
    
        ```
                class VendAccountNumRunnableJob
                {
                    public static void main(Args _args)
                    {
                        // Create an instance of the VendAccountNum class
                        VendAccountNum vendNumSeq = new VendAccountNum();
                
                        // Load the number sequence module by calling the load method
                        vendNumSeq.load();
                    }
                }
      ```
#### Step 4: 
  - This step involves setting the number sequence reference to a number sequence code which defines the format of the number sequence.
  - In the number sequence form (Organization Administration Module). Then click on generate for it to generate all the number sequence that are missing
  - To verify that they have been created, using the filters, you can select the reference to the EDT we created for this number sequences

#### Step 5: 
  - This step involves OverRiding the create method in the form with the field .
  - The method finds the number sequence reference associated with the EDT for that field.
  - Then the code calls the ‘num()‘ method to get the next number.
  - Finally, the code assigns that value to the newly created record on the data source.
  - The code for this can be found in the the file (VendAccountForm.xpp)
  - Or rather here is the code:

      ```
          // Override the create method
          public void create(boolean _append = false)
          {
              VendNumEDT vendNumEDT;
              NumberSequenceReference numberSequenceReference;
          
              super(_append);
          
              // Find the number sequence reference based on the extended data type (VendNumEDT)
              numberSequenceReference = NumberSeqReference::findReference(extendedTypeNum(VendNumEDT));
              
              // Check if a number sequence reference is found
              if (numberSequenceReference)
              {
                  // Generate the next number from the number sequence
                  vendNumEDT = NumberSeq::newGetNum(numberSequenceReference).num();
                  
                  // Assign the generated number to the vendor account number field
                  vendAccountTable.vendAccountNum = vendNumEDT;
              }
          }
    ```

  - And that's it. 
  


