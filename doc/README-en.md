[English Docs](README-en.md)  | [中文文档](../README.md)

#  Bluetooth Scale SDK

##  Ⅰ . Integration method - two methods
#####  gradle Automatic import method

1、Add build.gradle in the project directory

    allprojects {
        repositories {
            、、、
             maven { url 'http://nexus.lefuenergy.com/repository/maven-public' }
        }
    }
2、Add to build.gradle under the module that needs to import the SDK //Please use different artifactId according to different branches，The format is: ppscale-branch name // The following is the integration method of the master branch, the corresponding so file has been integrated
    
    dependencies {
        、、、
        implementation 'com.peng.ppscale:ppscale-new-master:0.0.4.4'     
    }
    
## Ⅱ .Instruction of use

* Due to the need the Bluetooth connection, Demo needs to run by a real machine.

* In Android 6.0 and above system versions, make sure to turn on it before start the scanning

    1、Location permission
    2、Positioning switch
    3、Bluetooth switch

* If you need the information other than the weight, you need to enter your height, age, gender and go on the scale with barefoot.

* Height range: 30-220 cm；Age range: 10-99 years old；Unit 0 represent kilogram，1 represent jin，2 represent pound；Gender 1 represent male，0 represent female；The value range of user group is 0-9（The specific scale needs this value）

* You need to turn on Bluetooth and give Demo location permission when you using the Demo

    1、Binding device - It will start scanning for nearby peripherals after this controller is instantiated and make a record of your peripherals.

    2、Weighing on scale - It will also start scanning nearby peripherals after this controller is instantiated, and connect to the bound devices through filtering. Therefore, the weighing can only be carried out after being bound, otherwise the data cannot be received.

    3、Equipment management - This controller will display the peripherals you bind on the "Bind Device" page in a list. You can delete the bound device by long pressing.

    4、After receiving the data returned by the peripherals on the "Bind Device" and "Weighing on Scales" pages, it will automatically stop scanning and disconnect from the peripherals, and then send the data back to the "Homepage Information" update via callback For the weight column, you can check the the specific data on "Data Details".

## Ⅲ .The use of ppscalelib
######  1.1 Bind or scan specified devices
    
        //The difference between binding device and scanning device is  searchType  0 Binding device 1 Scanning specified device
        //setProtocalFilterImpl() Receive data callback interface, process data, lock data, historical data
        //setDeviceList()The parameter of the function is not null，Binding needs to pass null (or not call)，Scan the specified device, please upload the list of the specified device<DeviceModel>。Bind
        //setBleOptions()Bluetooth parameter configuration
        //setBleStateInterface() need parameter PPBleStateInterface，Bluetooth status monitoring callback and system Bluetooth status callback    //startSearchBluetoothScaleWithMacAddressList（）Start scanning device
        //setUserModel() parameter PPUserModel normal status， setUserModel() is necessary，
        
        /**
        * sdk entrance, Instance object 
        */
       private void bindingDevice() {
               if (searchType == 0) {
                   ppScale = new PPScale.Builder(this)
                           .setProtocalFilterImpl(getProtocalFilter())
                           .setBleOptions(getBleOptions())
       //                    .setDeviceList(null)
                           .setBleStateInterface(bleStateInterface)
                            .setUserModel(userModel);
                           .build();
                   ppScale.startSearchBluetoothScaleWithMacAddressList();
               } else {
                   List<DeviceModel> deviceList = DBManager.manager().getDeviceList();
                   List<String> addressList = new ArrayList<>();
                   for (DeviceModel deviceModel : deviceList) {
                       addressList.add(deviceModel.getDeviceMac());
                   }
                   ppScale = new PPScale.Builder(this)
                           .setProtocalFilterImpl(getProtocalFilter())
                           .setBleOptions(getBleOptions())
                           .setDeviceList(addressList)
                           .setBleStateInterface(bleStateInterface)
                           .setUserModel(userModel)
                           .build();
                   ppScale.startSearchBluetoothScaleWithMacAddressList();
               }
           
Note: If you need to automatically cycle scan, you need to call again after loadedData() ppScale.startSearchBluetoothScaleWithMacAddressList()

######  1.2 BleOptions Bluetooth parameter configuration

        //Configure the type of scale that needs to be scanned default all，Optional 
     *                     Capabilities：
     *                     weighing scale {@link BleOptions.ScaleFeatures#FEATURES_WEIGHT}
     *                     Fat scale {@link BleOptions.ScaleFeatures#FEATURES_FAT}
     *                     Heart rate scale {@link BleOptions.ScaleFeatures#FEATURES_HEART_RATE}
     *                     Offline scale{@link BleOptions.ScaleFeatures#FEATURES_HISTORY}
     *                     Closed eye single foot scale{@link BleOptions.ScaleFeatures#FEATURES_BMDJ}
     *                     Weighing calculation {@link BleOptions.ScaleFeatures#FEATURES_CALCUTE_IN_SCALE}
     *                     WIFI scale {@link BleOptions.ScaleFeatures#FEATURES_CONFIG_WIFI} 请参考{@link BleConfigWifiActivity}
     *                     Food scale {@link BleOptions.ScaleFeatures#FEATURES_FOOD_SCALE}
     *                     All body scales {@link BleOptions.ScaleFeatures#FEATURES_NORMAL}  //Does not include food scale
     *                     All scales {@link BleOptions.ScaleFeatures#FEATURES_ALL}
     *                     customize {@link BleOptions.ScaleFeatures#FEATURES_CUSTORM} //If you choose to customize, you need to set PPScale.java-->setDeviceList()
               
        setFeaturesFlag(BleOptions.ScaleFeatures.FEATURES_NORMAL)
        
###### 1.3 PPBleStateInterface, Bluetooth status monitoring callback and system Bluetooth status callback

    //Contains two callback methods, one is Bluetooth status monitoring, the other is system Bluetooth callback

     PPBleStateInterface bleStateInterface = new PPBleStateInterface() {
            //Bluetooth status monitoring
            @Override
            public void monitorBluetoothWorkState(PPBleWorkState ppBleWorkState) {
                if (ppBleWorkState == PPBleWorkState.PPBleWorkStateConnected) {
                    Logger.d("Device connected");
                } else if (ppBleWorkState == PPBleWorkState.PPBleWorkStateConnecting) {
                    Logger.d("Device connecting");
                } else if (ppBleWorkState == PPBleWorkState.PPBleWorkStateDisconnected) {
                    Logger.d("Device disconnected");
                } else if (ppBleWorkState == PPBleWorkState.PPBleWorkStateStop) {
                    Logger.d("Stop scanning");
                } else if (ppBleWorkState == PPBleWorkState.PPBleWorkStateSearching) {
                    Logger.d("Scanning");
                } else {
                    Logger.e("Bluetooth status is abnormal");
                }
            }

        //system Bluetooth callback
        @Override
        public void monitorBluetoothSwitchState(PPBleSwitchState ppBleSwitchState) {
            if (ppBleSwitchState == PPBleSwitchState.PPBleSwitchStateOff) {
                Logger.e("System Bluetooth disconnect");
                Toast.makeText(BindingDeviceActivity.this, "System Bluetooth disconnect", Toast.LENGTH_SHORT).show();
            } else if (ppBleSwitchState == PPBleSwitchState.PPBleSwitchStateOn) {
                Logger.d("System Bluetooth turn on");
                Toast.makeText(BindingDeviceActivity.this, "System Bluetooth disconnect", Toast.LENGTH_SHORT).show();
            } else {
                Logger.e("System Bluetooth is abnormal");
            }
        }
    };

###### 1.4 ProtocalFilterImpl

    //Implement the interfaces according to requirements
    //Monitor process data setPPProcessDateInterface()
    //Monitor lock data setPPLockDataInterface()
    //Monitor history data setPPHistoryDataInterface()
   
     ProtocalFilterImpl protocalFilter = new ProtocalFilterImpl();
            protocalFilter.setPPProcessDateInterface(new PPProcessDateInterface() {
            //process data
                @Override
                public void monitorProcessData(PPBodyBaseModel bodyBaseModel) {
                    Logger.d("bodyBaseModel scaleName " + bodyBaseModel.getScaleName());
                    //weight
                    String weightStr = PPUtil.getWeight(unitType, bodyBaseModel.getPpWeightKg());
            }
        });
        protocalFilter.setPPLockDataInterface(new PPLockDataInterface() {
            //Monitor lock data
            @Override
            public void monitorLockData(PPBodyFatModel bodyFatModel, PPDeviceModel deviceModel) {  
                if (bodyFatModel.isHeartRateEnd()) {
                    if (bodyFatModel != null) {
                        Logger.d("monitorLockData  bodyFatModel weightKg = " + bodyFatModel.getPpWeightKg());
                    } else {
                        Logger.d("monitorLockData  bodyFatModel heartRate = " + bodyFatModel.getPpHeartRate());
                    }
                    String weightStr = PPUtil.getWeight(unitType, bodyFatModel.getPpWeightKg());
                    if (weightTextView != null) {
                        weightTextView.setText(weightStr);
                        showDialog(deviceModel, bodyFatModel);
                    }
                } else {
                    Logger.d("Measuring the Heart Rate");
                }
            }
        });

        if (searchType != 0) {
            //Do not receive offline data when binding the device， If you need to receive offline data, please implement this interface
            protocalFilter.setPPHistoryDataInterface(new PPHistoryDataInterface() {
                @Override
                public void monitorHistoryData(PPBodyFatModel bodyBaseModel, boolean isEnd, String dateTime) {
                    if (bodyBaseModel != null) {
                        Logger.d("ppScale_ isEnd = " + isEnd + " dateTime = " + dateTime + " bodyBaseModel weight kg = " + bodyBaseModel.getPpWeightKg());
                    } else {
                        Logger.d("ppScale_ isEnd = " + isEnd);
                    }
                }
            });
        }
     

###### 1.5 PPUserModel Basic information of user
    
    userHeight、age、sex Must be true
    userHeight range is 100-220cm
    age range is10-99
    sex 0 is female 1 is male   
    
###### 1.6 Related Bluetooth Operation

Reserved Bluetooth operation object

    BluetoothClient client = ppScale.getBleClient();
    
Stop scanning

    ppScale.stopSearch();
   
Disconnect device

    ppScale.disConnect();
    
Finally you need to call the stopSearch method before leaving the page. For specific implementation, please refer to the code in BindingDeviceActivity and ScaleWeightActivity in Demo。

###### 1.7 PPBodyFatModel Parameter Description

    protected int impedance;                                //Impedance value (encrypted)
    //    protected float ppZTwoLegs;   //Foot-to-foot impedance value(Ω), range 200.0 ~ 1200.0
    protected double ppWeightKg;                                              //Weight
    protected int ppHeartRate;                                              //Heart Rate
    protected int scaleType;                                                //Scale’s type
    protected boolean isHeartRateEnd = true;                            //Heart rate end icon
    protected String scaleName;                                            //Scale’s name
    
    protected PPUserModel userModel;
    protected PPUserSex ppSex;                                                //Gender
    protected double ppHeightCm;                  //Height(cm)，need to between 90 ~ 220cm
    protected int ppAge;                     //age(years old), need to between 6 ~ 99 years old
    protected double ppProteinPercentage;          //protein, Resolution0.1, range 2.0% ~ 30.0%
    protected int ppBodyAge;                             //body age, 6~99 years old protected double ppIdealWeightKg;                                   //Ideal weight(kg)
    protected double ppBMI;            //BMI Body mass index, Resolution 0.1, range 10.0 ~ 90.0
    protected int ppBMR;           //Basal Metabolic Rate BMR, Resolution 1, range 500 ~ 10000
    protected int ppVFAL;          //Visceral fat area leverl Visceral fat, Resolution 1, range 1 ~ 60
    protected double ppBoneKg;                //bone mass(kg), Resolution 0.1, range 0.5 ~ 8.0
    protected double ppBodyfatPercentage;       //fat rate(%), Resolution 0.1, range5.0% ~ 75.0%
    protected double ppWaterPercentage;        //water(%), Resolution 0.1, range 35.0% ~ 75.0%
    protected double ppMuscleKg;          //Muscle mass(kg), Resolution 0.1, range 10.0 ~ 120.0
    protected int ppBodyType;                                               //body type
    protected int ppBodyScore;                                 //Body score 50 ~ 100 points
    protected double ppMusclePercentage;       //Muscle rate(%),Resolution 0.1，range 5%~90%
    protected double ppBodyfatKg;                                          //Fat mass(kg)
    protected double ppBodystandard;                                //Standard Weight(kg)
    protected double ppLoseFatWeightKg;                              //Lean body mass(kg)
    protected double ppControlWeightKg;                               //Weight control(kg)
    protected double ppFatControlKg;                                 //Fat control mass(kg)
    protected double ppBonePercentage;                              //Bone Percentage(%)
    protected double ppBodyMuscleControlKg;                      //Muscle control mass(kg)
    protected double ppVFPercentage;                                //Subcutaneous fat(%)
    protected PPBodyEnum.PPBodyGrade ppBodyHealth;                            //Body Health
    protected PPBodyEnum.PPBodyFatGrade ppFatGrade;                           //Fat Grade
    protected PPBodyEnum.PPBodyHealthAssessment ppBodyHealthGrade;            //Body Health Grade
    protected PPBodyEnum.PPBodyfatErrorType ppBodyfatErrorType;                //Error type

Note: When you get the object when using it, please call the corresponding get method to get the corresponding value

###### 1.8  PPBodyFatModel Parameter Description
  
  1.Error type: PPBodyEnum.PPBodyfatErrorType
  
      PPBodyfatErrorTypeNone(0),          //!< No error (all parameters can be read)
      PPBodyfatErrorTypeImpedance(-1),    //!< If the impedance is wrong, when the impedance is wrong, the parameters other than BMI/idealWeightKg will not be calculated (write 0)
      PPBodyfatErrorTypeAge(-1),          //!< The age parameter is wrong, and it needs to be between 6 and 99 years old (not counting parameters other than BMI/idealWeightKg)
      PPBodyfatErrorTypeWeight(-2),       //!< The weight parameter is wrong, it needs to be 10 ~ 200kg (all parameters will not be calculated if it is wrong)
      PPBodyfatErrorTypeHeight(-3);       //!< The height parameter is wrong, it needs to be 90 ~ 220cm (not counting all parameters)

  2.Body Health: PPBodyEnum.PPBodyfatErrorType
  
      PPBodyGradeThin(0),             //!< Lean
      PPBodyGradeLThinMuscle(1),      //!< Standard
      PPBodyGradeMuscular(2),         //!< Super heavy
      PPBodyGradeLackofexercise(3);   //!< Obese

  3.肥胖等级 PPBodyEnum.PPBodyfatErrorType
  
      PPBodyGradeFatOne(0),             //!< Obesity Grade 1
      PPBodyGradeLFatTwo(1),            //!< Obesity Grade 2
      PPBodyGradeFatThree(2),           //!< Obesity Grade 3
      PPBodyGradeFatFour(-1);           //!< Parameter error
  
  4.健康等级 PPBodyEnum.PPBodyfatErrorType
  
      PPBodyAssessment1(0),             //!< Health risks
      PPBodyAssessment2(1),             //!< Sub-health
      PPBodyAssessment3(2),             //!< general
      PPBodyAssessment4(3),             //!< good
      PPBodyAssessment5(4),             //!< very good
      PPBodyAssessmentError(-1);        //!< Parameter error

## IV .ppscalelib在WIFI设备的使用

###### 2.0 蓝牙配网

Specific reference：{@link BleConfigWifiActivity}

        ProtocalFilterImpl protocalFilter = new ProtocalFilterImpl();
        //监听配网结果 setConfigWifiInterface()
        protocalFilter.setConfigWifiInterface(new PPConfigWifiInterface() {
            
            /**
             * wifi设备配网成功并获取到SN
             *
             * @param sn 设备识别码
             */
             @Override
             public void monitorConfigState(String sn) {
                 //拿到sn 处理业务逻辑
                 Logger.e("xxxxxxxxxxxx-" + sn);
             }
         });
        
        ppScale = new PPScale.Builder(this)
                   .setProtocalFilterImpl(protocalFilter)
                   .setBleOptions(getBleOptions())
                   .setBleStateInterface(bleStateInterface)
                   .build();
        ppScale.startSearchBluetoothScaleWithMacAddressList();

WIFI parameter configuration

        /**
            * 参数配置 绑定时请确保WIFI是2.4G，并且账号密码正确
            *
            * @param password     WIFI密码
            * @param featuresFlag 具备的能力，WIFI秤{@link BleOptions.ScaleFeatures#FEATURES_CONFIG_WIFI}                      
            * @parm ssid          WIFI账号  不可为空
            * @return
            */
           private BleOptions getBleOptions() {
               return new BleOptions.Builder()
                       .setFeaturesFlag(BleOptions.ScaleFeatures.FEATURES_CONFIG_WIFI)
                       .setPassword("12345678")
                       .setSsid("IT05-2.4G")
                       .build();
           }

Bluetooth status monitoring, please refer to this document:   1.3  PPBleStateInterface       


## V .Version update instructions
   
    ----0.0.1-----
    1、Add maven configuration  2、Increase compatibility 'BodyFat Scale1'
    ----0.0.2-----
    1、Add Bluetooth WIFI distribution network function
    ----0.0.3-----
    1、Optimize Bluetooth distribution network function  2、Improve broadcast data compatibility


Contact Developer：
Email: yanfabu-5@lefu.cc

   
   
   
