

# Aradas L&L Fields and Variables

## Test case 
Test case could be found at https://stg.prefweb.com/Aradas/PrefWeb/SalesDocuments/Edit?number=101599&version=1

 

## QID Texts
Feature  allows to 'cook' custom 'SalesDocItem' descripotion.  Grouping, sorting and formating base material and/or material `DesAuto`  and P-Options values `DesAuto` to description blocks. Those description blocks defines by `DesAutoOrden` . Each bloc has `DesAutoOrden` start and end indexes. Each block define and set L&L field for printouts. That allow more flexible agreagate sales document item descriptions. Distinct each group in pribntouts 
- **Module**: SalesModule
- **Event**: Sales_DefineVariablesAndFields , Sales_SetFields
- **Conditiios**: Global Variable `QIDEnabled` = 1 
- **Settings**: settings stored at `dbo.UniwaveQID` and  dbo.`Uniwave_QIDNumericOptions` tables

1. Materials and non-alaphanumeric option values

    Settings: `dbo.UniwaveQID`
     ```sql
     SELECT [RowId]  -- unique Id 
           ,[BlockName] -- L&L Varaiable name (QID.<BlockName>) 
           ,[BlockDesc] -- Description used at UI
           ,[PrintFlag] -- Print: 1=Yes, 0=No (Aradas allways1)
           ,[BlockType] -- Type: 0=Materials, 1=Options
           ,[BlockStart] -- Orden DesAuto (MaterialesBase, Materiales, OptionValor)
           ,[BlockEnd] -- Orden DesAuto (MaterialesBase, Materiales, OptionValor)
           ,[BlockFields] -- 0 dont show, 1 = show sepaarte line ,2 = ??? , 3 = grouped single line using ','  
           ,[BlockOption] -- define how text is separated within block: 0='\n', 1=' ', 2=','  
           ,[FieldsOption] -- not used in Aradas
           ,[GlassDimensions]  -- not used in Aradas
       FROM [dbo].[UniwaveQID]
     
     ```

     | RowId                                | BlockName     | BlockDesc            | PrintFlag | BlockType | BlockStart | BlockEnd | BlockFields | BlockOption | FieldsOption | GlassDimensions |
     | ------------------------------------ | ------------- | -------------------- | --------- | --------- | ---------- | -------- | ----------- | ----------- | ------------ | --------------- |
     | B5B455F1-DF69-4616-845B-081F9B5D97E0 | GlassUnits    | Glass Units          | 1         | 0         | 1500       | 1799     | 3           | 0           | 0            | NULL            |
     | 072CE647-666A-4F3B-A030-0CD08B6D7A3B | QID_TR        | Transport K          | 1         | 0         | 2000       | 2000     | 3           | 3           | 0            | NULL            |
     | 879DB4A1-206D-4403-8B14-1AB7745BC90A | Threshold     | MainProfileThreshold | 1         | 0         | 650        | 655      | 0           | 2           | 0            | NULL            |
     | E8D69E6E-DE60-4362-B53F-1C71B23A469F | HingeColor    | Hinge Color          | 1         | 0         | 9080       | 9080     | 0           | 2           | 0            | NULL            |
     | 7C827DE8-589A-4491-90AC-205898BDFE4C | OpeningType   | Opening Type         | 1         | 0         | 9003       | 9003     | 0           | 2           | 0            | NULL            |
     | 9FC937E0-CDAA-4748-AEB1-21FB6190BAA1 | NLSnapAround  | NLSnapAround         | 1         | 0         | 770        | 774      | 0           | 2           | 0            | NULL            |
     | 9C85ADC6-BEBB-4BD4-BD9F-2349897CA59E | GlazingBeads  | Glazing beads        | 1         | 0         | 3000       | 3499     | 1           | 2           | 0            | NULL            |
     | 39DD6E24-054B-4E4C-9BCE-28B901FFD112 | RensonControl | RensonControl        | 1         | 0         | 3530       | 3540     | 0           | 3           | 0            | NULL            |
     | A242CADD-9398-47EA-81F3-28C967F87F43 | MountingSides | Mounting sides       | 1         | 0         | 7501       | 7504     | 2           | 2           | 0            | NULL            |

 1. Alphanumeric option values 

      Settings: `dbo.Uniwave_QIDNumericOptions`
      ```sql
      SELECT [RowId]
         ,[OptionName]
         ,[BlockOrden]
         ,[Settings]
      FROM [Aradas].[dbo].[Uniwave_QIDNumericOptions]
      ```

     | RowId                                | OptionName                | BlockOrden | Settings |
     | ------------------------------------ | ------------------------- | ---------- | -------- |
     | D32CDB51-1715-4BEB-8B9C-22309E1262B1 | Mullion Color             | 1843       | 1        |
     | CE2B12C0-E970-460A-9C4C-2D9FB8EDDA84 | Model Color               | 1840       | 1        |
     | E5E9E7B3-0868-4A02-96DA-3D71ED0D08E4 | Sash Mullion Color        | 1844       | 1        |
     | DEA74496-D4F7-4240-991E-3F1848E8F8E2 | Manual Glass Description  | 1690       | 1        |
     | CFAB4A83-9F6F-49B5-B9FD-43C70087AAB1 | Sash Stop Color           | 1845       | 1        |
     | 6182BA50-BD85-4367-91D2-562BFDB51D77 | Sash Color                | 1842       | 1        |
     | 19931563-A3EE-4C0E-884A-979E2F5188BE | FrameQID                  | 805        | 1        |
     | 0AEE3057-8B73-40B5-B788-DDA1AE98254C | MV_ProfileDepth           | 807        | 1        |
     | 5030D742-15C4-470B-B4E7-E53C8997C15D | Commercial Offer Comments | 9991       | 1        |
     | 5D435485-A27A-484A-908F-F6B7960169DE | Frame Color               | 1841       | 1        |

- **Source of data** : materials  details (references, descriptions, colors, dimensions), options, optionvalues text, fields numbers of materials and other data extarcted from `Interop.PrefSales.SalesDocItem.DescriptiveXML`

- **Multilinguality** : used language of sales document also   getting collection of sales document additional languages `Interop.PrefSales.SalesDoc.AdditionalLanguages`. Translated values of materials and options values are queried from `dbo.LanguageContent` table. 

- **Data stored**: Generated text is cached and stored at table `dbo.Uniwave_ContenidoPAFDescriptions`. Data generated and stored eche time on  event ` Sales_DefineVariablesAndFields`. DeaAuto text it order and translations are cached, so database is queried just in case   

  ```sql
  SELECT [Number]
      ,[Version]
      ,[Position]
      ,[LanguageId]
      ,[QIDRowId]
      ,[Description]
  FROM [dbo].[Uniwave_ContenidoPAFDescriptions]
  ```


    | Number | Version | Position | LanguageId | QIDRowId                             | Description                                                  | 
    | ------ | ------- | -------- | ---------- | ------------------------------------ | ------------------------------------------------------------ | 
    | 102942 | 1       | 1        | 1030       | B5B455F1-DF69-4616-845B-081F9B5D97E0 | 1,2: Energi XN, 4-16Ar-4 XN(24/2L),  Ug=1.12, g=65%, TL=82%, Rw=31(-1, -4) 3,4,5: PVC Filling 24mm |     
    | 102942 | 1       | 1        | 1033       | 847348CD-3201-4B02-8808-815AD47DE804 | London                                                       |     
    | 102942 | 1       | 1        | 1033       | FC74973F-B1B1-461C-9CAB-953ADE25517F | outside                                                      |     
    | 102942 | 1       | 1        | 1033       | 8F087EE9-DD91-4B23-BAAD-A1C3EFBDADB5 | T105mm                                                       |     
    | 102942 | 1       | 1        | 1033       | 5CA0AB21-3FB8-442F-9AAC-A5C729D1F1F1 | MILA                                                         |     
    | 102942 | 1       | 1        | 1033       | 5183A5C9-3939-49AF-BC38-B7D5D2DDCDD9 | no                                                           |     
    | 102942 | 1       | 1        | 1033       | B9496F23-1B4B-4CDA-B095-BBE9696AE551 | aluminium                                                    |     
    | 102942 | 1       | 1        | 1033       | CBE9B860-A923-40A3-8EA4-BE70D3725AA8 | vWhite, GG / vWhite, GG                                      |     
    | 102942 | 1       | 1        | 1033       | D2685C43-2CAD-4F3E-B324-C10C54CA6C28 | without                                                      |     
    | 102942 | 1       | 1        | 1033       | 5D7BBEAA-A54B-46C0-AC97-C1125153EE92 | outward opening entrance door London  white MILA silver key/key (wilka) |     
    | 102942 | 1       | 1        | 1033       | 5D5B24CF-4BDE-4860-AE97-CF1DE410F2C6 | HIMALO / REHAU Nordic Design Plus  entrance door             |      
    | 102942 | 1       | 1        | 1033       | 87364E68-E37D-415D-BE99-D4D785FD98DF | RAL9016                                                      |      
    | 102942 | 1       | 1        | 1033       | A13EB6DA-E497-408F-AC4F-DA6341E08117 | 42x120mm                                                     |      
    | 102942 | 1       | 1        | 1033       | 64F092F3-9676-43C5-A352-DAFD7804C0EB | duplex                                                       |      
    | 102942 | 1       | 1        | 1033       | 2A72FD36-C72C-4758-B6D4-DBD0820EF513 | no                                                           |      
    | 102942 | 1       | 1        | 1033       | ECBA80DC-919C-4BFE-B02B-F149B52D6DFD | white                                                        |      
    | 102942 | 1       | 1        | 1033       | 44403D0C-293B-4BDB-BED9-F9A7964309F9 | vWhite, GG / vWhite, GG                                      |      
    | 102942 | 1       | 1        | 1033       | 95794CE1-3580-4D11-92C7-FDE52C8EF0A8 | RAL7016                                                      |      
    | 102942 | 1       | 1        | 1043       | B5B455F1-DF69-4616-845B-081F9B5D97E0 | 1,2: Energieklasse XN, 4-16Ar-4  XN(24/2L), Ug=1.12, g=65%, TL=82%, Rw=31(-1, -4) 3,4,5: PVC Filling 24mm |     
    | 102942 | 1       | 1        | 1043       | D8472C0C-B252-4217-A3D8-0F7C75E06B5C | 120                                                          |     
    | 102942 | 1       | 1        | 1043       | 879DB4A1-206D-4403-8B14-1AB7745BC90A | aluminium 24.7mm                                             |     
    | 102942 | 1       | 1        | 1043       | E8D69E6E-DE60-4362-B53F-1C71B23A469F | zilver                                                       |     
    | 102942 | 1       | 1        | 1043       | 7C827DE8-589A-4491-90AC-205898BDFE4C | buitendraaiende entree deur                                  |     
    | 102942 | 1       | 1        | 1043       | 9C85ADC6-BEBB-4BD4-BD9F-2349897CA59E | hoekig                                                       |     
    | 102942 | 1       | 1        | 1043       | BD576319-5ED7-4309-86FE-2FA4C05389FC | 2x koppelings dichting                                       |     
    | 102942 | 1       | 1        | 1043       | 2842DB90-C278-47D7-B388-39F6071FD7D2 | nee                                                          |     
    | 102942 | 1       | 1        | 1043       | 2F8C50F6-4C96-417D-A4CC-3AD61246F423 | 61x103mm decoratief                                          |     
    | 102942 | 1       | 1        | 1043       | 9B30A95D-847F-44F8-8D4A-3B333E0DA589 | nee                                                          |     
    | 102942 | 1       | 1        | 1043       | 9D858122-35D7-4916-8593-4360E9728537 | vWhite, GG / vWhite, GG                                      |     
    | 102942 | 1       | 1        | 1043       | AD796326-56B2-463A-BF1D-4A969377C2E2 | 61mm                                                         |     
    | 102942 | 1       | 1        | 1043       | 35D8410E-0DF2-4329-90DC-5503351C4234 | sleutel/sleutel(wilka)                                       |     
    | 102942 | 1       | 1        | 1043       | 0B2DD262-89C6-4562-A870-566B33547736 | click-vent 344x27 wit                                        |     
    | 102942 | 1       | 1        | 1043       | CD2E1CCE-1D26-4767-B128-58800C255772 | RAL7016                                                      |     
    | 102942 | 1       | 1        | 1043       | 2C079737-B60A-427B-98D6-5A7BC01C1C7C | standaard 27x8mm                                             |     


## Varaiables using custom user SQL Queries

- **Module**: SalesModule
- **Event**: Sales_DefineVariablesAndFields, Sales_SetVariables
- **Variables**: User.PhonePersonalMobile, User.Fax
- **Settings:** stored in database tables `Uniwave_CustomReportsQuerys` and `Uniwave_CustomReportsQueryList`

     ```SQL
     SELECT [RowId]
           ,[Name] -- 
           ,[Kind] -- for example SalesDoc
           ,[Document] -- ame of print template layout 
           ,[QueryRowId]
           ,[Type] -- 1 = variabales, -- 2 = fields 
           ,[SortOrder]
           ,[Drawing]
           ,[XmlSource]
       FROM [Aradas].[dbo].[Uniwave_CustomReportsQuerys]
       Where kind ='SalesDoc' and [Name] = '00 - Commercial Document'
     ```

    | RowId                                | Name                     | Kind     | Document | QueryRowId                           | Type | SortOrder | Drawing | XmlSource |
    | ------------------------------------ | ------------------------ | -------- | -------- | ------------------------------------ | ---- | --------- | ------- | --------- |
    | 4D00B23C-04F9-48F1-ABC6-6A264B99C177 | 00 - Commercial Document | SalesDoc | NULL     | B3D12B7A-892F-42B5-9245-C92029A604AD | 1    | NULL      |         | NULL      |
    | 8ACD4901-1106-46EC-8A45-F0FF159136C8 | 00 - Commercial Document | SalesDoc | NULL     | D3427D1F-F319-488D-A433-08A0F5773687 | 1    | NULL      |         | NULL      |


  ```sql 
  SELECT  [RowId] -- unique Id of querie
         ,[Name] -- querie name
         ,[Kind] -- 1 = Varaiabels, 2 =  fields 
         ,[Body]
         ,[SortOrder]
         ,[Type]
         ,[Namespace]
  FROM [dbo].[Uniwave_CustomReportsQueryList]
  ```

    | RowId                                | Name           | Kind | Body                                                         | SortOrder | Type | Namespace |
    | ------------------------------------ | -------------- | ---- | ------------------------------------------------------------ | --------- | ---- | --------- |
    | D3427D1F-F319-488D-A433-08A0F5773687 | Transport      | 1    | ```sql SELECT      SUM(Amount) [Transport.Amount],     (SELECT ISNULL([dbo].[Uniwave_GetTransportDistr]      ({Number},{Version}),N'No')) [Transport.PreSetting]     FROM dbo.vwSalesDetail WHERE System=N'Transport' AND Number={Number} AND  Version={Version}``` | 56        |     1    | NULL      |
    | B3D12B7A-892F-42B5-9245-C92029A604AD | Transport Type | 1    | ```sql SELECT ISNULL([dbo].[Uniwave_GetTransportType]  ({Number},{Version}),N'NoPresettings')     [TransportType.PreSetting]``` | 92        | 1    | NULL      |

> Queries results mandatory values
> - Variabeles - should contain columns `Number`, `Version`,
> - Fields  - should contain columns `Number`, `Version`, `Orden`
> - Values of rest columns returned by SQL query will be used as variables or fields in L&L print layout. 

  - **Data stored:** fileds or/and varaiables are not stored. Defined and set just on events execution. 
----

## Other Fields & Variables 
Exists set of fields that are hardcoded in Uniwave code and used as L&L fields in Aradas sales reports fields like: `TEXTO` , `PrtoductType` and others. Varaiables are takend from Extarcted from `Interop.PrefSales.SalesDoc` and/or `Interop.PrefSales.SalesDocItem` properties.  

- **Module:** SalesModule
- **Event:** Sales_DefineVariablesAndFields, Sales_SetFields
- **Settings:** not exists.
- **Data stored:** fileds or/and varaiables are not stored. Defined and set just on events execution. 
> All varaiables or fields are calculated for individual `SalesDocItem`. No aggregations or values related to other `SalesDocItem` or `SalesDoc`.
---

## Model Variables ##
>Could be fully replaced by kernel. Should be used feature where we can flag required `Model Varaiable` as available for `Sales printout`. 
---

## PDG Price Document Groups 
>Not used in Aradas 
---

# Other related topics 

## Transportation
Special development which add `SalesDocItem` as last position in `SalesDoc`. Calculation made based on `SalesDoc` presettings parameters choosen at **Menu for User**. Added  `SalesDocItem` named as "LDM" (Loading Meters). Calculation done using complex calculation, that calculate packing (plets) and transport price based on set `SalesDocItem` properties like width, height, thicknes, weight and others.
  
- **Module:** SalesModule
- **Event:** SalesDoc_BeforeSave, SalesDoc_Validate 
- **Settings:** collection of cusom tables
  
### Calculation 
- User presettings choice 

    ![Presettings ](../aradas-l-and-l/Images/presettings.png)

- Added Position at sales document 
  
   ![Position](../aradas-l-and-l/Images/ldm-position.png)

> Calculation executed once per sales document on event `SalesDoc_BeforeSave`. **Not** execute after `SalesDocItem` are added. Its last step before user printing quote or order printout. 

### Validation 

Validate sales document and output errors at to `Validation Messages`
![Validation](../aradas-l-and-l/Images/validation.png)
> Validation executed per sales document on event `SalesDoc_Validate`. **Not** execute after `SalesDocItem` 
added. Its last step before user printing quote or order printout. 
  
- **Data stored:** stored as `SalesDocItem` in `SalesDoc`.
  
---  
## Profiles Optimization Remnants
Currently not used, but exisats in Aradas roadmap. Feature for calculating remnants cost in sales documents using   `Preference.Customization.SignedRetales.Retales.CalculoRetales` . 