# RSB_Inward-Gatepass-Report
Inward Gatepass Report

Service Definitions 

@EndUserText.label: 'Gate Pass Details Service Definition'
define service ZUI_GATEPASSHEADER_M {
  expose ZC_GATEPASSHEADER_M;
  expose ZC_GATEPASSITEM_M;
}
******************************************************************
@EndUserText.label: 'Service definition for ZC_GPCOUNTER'
define service ZUI_GPCOUNTER_O4 {
  expose ZC_GPCOUNTER;
}
***************************************************

Behaviour Definitions 

projection;
strict ( 2 );
use draft;
use side effects;


define behavior for ZC_GATEPASSHEADER_M alias header
{
  use create;
  use update;
  use delete;

  use action Resume;
  use action Edit;
  use action Activate;
  use action Discard;
  use action Prepare;
  //  use action other;
  use association _ITEM { create; with draft; }

}

define behavior for ZC_GATEPASSITEM_M alias item
{
  use update;
  use delete;

  use association _HEADER { with draft; }
}
***********************************************************

managed implementation in class zbp_i_gtpassheader_m unique;
strict ( 2 );
with draft;

define behavior for ZI_GTPASSHEADER_M alias header
persistent table zgatepassheader
draft table zgatepassheaderd
lock master
total etag LastChangedAt
authorization master ( instance )
etag master LocalInstanceLastChangedAt
{
  create; //( precheck );
  update; //( precheck );
  delete;
  association _ITEM { create ( authorization : update ); with draft; }
  field ( numbering : managed, readonly ) headeruuid;
  field ( readonly ) Gatepassno, vendoraddress, Gatepassdatetime, gatepassdate;
  field ( mandatory ) Plant, inv_verified, is_verified, Vendorcode;
  //  field ( features : instance ) oth_verified;

  draft action Resume with additional implementation;
  draft action Edit with additional implementation;
  draft action Activate optimized with additional implementation;
  draft action Discard;

  //  determination otherVerified on modify { field oth_verified; }
  determination setGatePassId on save { create; }
  determination getVendorAddress on modify { field Vendorcode; }


  validation validateplant on save { create; update; field Plant, inv_verified, is_verified; }
  side effects
  {
    field Vendorcode affects field vendoraddress;
  }

  draft determine action Prepare
  {
    validation validateplant;
    validation ZI_GTPASSITEM_M~validatefields;
    validation ZI_GTPASSITEM_M~validategatepass;
  }

  mapping for zgatepassheader corresponding
    {
      Headeruuid                 = headeruuid;
      gatepassno                 = gatepassno;
      gatepassdatetime           = gatepassdatetime;
      vendorcode                 = vendorcode;
      vendoraddress              = vendoraddress;
      lrnumber                   = lrnumber;
      lrdate                     = lrdate;
      plant                      = plant;
      vechiletype                = vechiletype;
      vechilecapacity            = vechilecapacity;
      vechilecnumber             = vechilecnumber;
      transporterid              = transporterid;
      lorryreciept               = lorryreciept;
      createdby                  = createdby;
      createdat                  = createdat;
      lastchangedby              = lastchangedby;
      lastchangedat              = lastchangedat;
      localinstancelastchangedat = local_instance_last_changed_at;
    }
}

define behavior for ZI_GTPASSITEM_M alias item
persistent table zgtpassitem
draft table zgatepassitemd
lock dependent by _HEADER
authorization dependent by _HEADER
etag master LastChangedAt
{

  update;
  delete;
  field ( numbering : managed, readonly ) Itemuuid;
  field ( readonly ) Headeruuid, Incoterms, Shipfrmloc, grnno, slno;
  //  field ( readonly : update ) PoNumber;
  field ( mandatory ) Ponumber, Invoicedate, Invoiceno;
  //  validation validatePonumber on save { create; update; field Ponumber; field Invoiceno; field Invoicedate; }
  validation validatefields on save { create; update; field Invoiceno; field Invoicedate; }
  validation validategatepass on save { create; update; field Invoiceno; field Invoicedate; }
  association _HEADER { with draft; }
  determination getserialnumber on modify { create; }
  determination getincotermsandshiploc on modify { field Ponumber; }
  //  determination touppercase on save { create; update; field Invoiceno; }
  //  validation touppercase on save { create; update; field Invoiceno; }
  side effects
  {
    field Ponumber affects field Incoterms, field Shipfrmloc, field grnno;
  }

  mapping for zgtpassitem corresponding
    {
      Itemuuid      = itemuuid;
      //      gatepassno    = gatepassno;
      irnnumber     = irnnumber;
      invoiceno     = invoiceno;
      invoicedate   = invoicedate;
      ponumber      = ponumber;
      irndate       = irndate;
      ewaybillno    = ewaybillno;
      ewaybilldate  = ewaybilldate;
      incoterms     = incoterms;
      shipfrmloc    = shipfrmloc;
      odnno         = odnno;
      grnno         = grnno;
      createdby     = created_by;
      createdat     = created_at;
      lastchangedby = last_changed_by;
      lastchangedat = last_changed_at;
    }
}
*************************************************************************************

Data Definitions 

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Gate Pass Header Consumption View'
@Metadata.ignorePropagatedAnnotations: true
@Metadata.allowExtensions: true
@Search.searchable: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define root view entity ZC_GATEPASSHEADER_M
  provider contract transactional_query
  as projection on ZI_GTPASSHEADER_M
{
  key Headeruuid,
      @Search.defaultSearchElement: true
      Gatepassno,
      Vendorcode,
      vendoraddress,
      Gatepassdatetime,
      Lrnumber,
      Lrdate,
      @Consumption.valueHelpDefinition: [{ entity: {name: 'I_PlantStdVH', element: 'Plant' }, useForValidation: true }]
      Plant,
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZI_VECHILETYP', element: 'VechType'} }]
      Vechiletype,
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZI_VECHILECAPACITY', element: 'Vechcap_des'} }]
      Vechilecapacity,
      Vechilecnumber,
      Transporterid,
      remarks,
      invremark,
      //      criticality_val,
      inv_verified,
      lorryreciept,
      is_verified,
      packlist,
      pack_verified,
      billofentry,
      boe_verified,
      other,
      oth_verified,
      gatepassdate,
      Createdby,
      Createdat,
      Lastchangedby,
      Lastchangedat,
      /* Associations */
      _ITEM : redirected to composition child ZC_GATEPASSITEM_M
}
*********************************************************************************************

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Gate Pass Item Consumption View'
@Metadata.ignorePropagatedAnnotations: true
@Metadata.allowExtensions: true
@Search.searchable: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZC_GATEPASSITEM_M
  as projection on ZI_GTPASSITEM_M
{
  key Itemuuid,
      Headeruuid,
      slno,
//      Gatepassno,
      Irnnumber,
      @Search.defaultSearchElement: true
      Invoiceno,
      Invoicedate,
      @Consumption.valueHelpDefinition: [{ entity: {name: 'ZI_PONUMBERVH', element: 'PurchasingDocument' }, useForValidation: true }]
      Ponumber,
      Irndate,
      Ewaybillno,
      Ewaybilldate,
      Incoterms,
      Shipfrmloc,
      odnno,
      grnno,
      CreatedBy,
      CreatedAt,
      LastChangedBy,
      LastChangedAt,
      /* Associations */
      _HEADER : redirected to parent ZC_GATEPASSHEADER_M
}
*************************************************************************

@AccessControl.authorizationCheck: #CHECK
@Metadata.allowExtensions: true
@EndUserText.label: 'Projection View for ZI_GPCOUNTER'
define root view entity ZC_GPCOUNTER
  provider contract transactional_query
  as projection on ZI_GPCOUNTER
{
  key UUID,
  Gpdate,
  Lastnum,
  LocalInstanceLastChangedAt
   
}
*******************************************************

@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: '##GENERATED ZGPCOUNTER'
define root view entity ZI_GPCOUNTER
  as select from zgp_counter
{
  key uuid as UUID,
  gpdate as Gpdate,
  lastnum as Lastnum,
  @Semantics.user.lastChangedBy: true
  last_changed_by as LastChangedBy,
  @Semantics.systemDateTime.lastChangedAt: true
  last_changed_at as LastChangedAt,
  @Semantics.systemDateTime.localInstanceLastChangedAt: true
  local_instance_last_changed_at as LocalInstanceLastChangedAt
   
}
******************************************************************

@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Gate Pass Header Root View'
@Metadata.ignorePropagatedAnnotations: true
@Metadata.allowExtensions: true
define root view entity ZI_GTPASSHEADER_M
  as select from zgatepassheader
  composition [0..*] of ZI_GTPASSITEM_M as _ITEM
{
  key headeruuid                     as Headeruuid,
      gatepassno                     as Gatepassno,
      vendorcode                     as Vendorcode,
      vendoraddress,
      gatepassdatetime               as Gatepassdatetime,
      lrnumber                       as Lrnumber,
      lrdate                         as Lrdate,
      plant                          as Plant,
      vechiletype                    as Vechiletype,
      vechilecapacity                as Vechilecapacity,
      vechilecnumber                 as Vechilecnumber,
      transporterid                  as Transporterid,
      remarks,
      invremark,
      //      case inv_verified
      //       when 'Yes' then 3 --Green
      //       when 'No' then 1 --red
      //       else 0
      //       end                           as criticality_val,
      inv_verified,
      lorryreciept,
      is_verified,
      packlist,
      pack_verified,
      billofentry,
      boe_verified,
      other,
      oth_verified,
      gatepassdate,
      @Semantics.user.createdBy: true
      createdby                      as Createdby,
      @Semantics.systemDateTime.createdAt: true
      createdat                      as Createdat,
      @Semantics.user.lastChangedBy: true
      lastchangedby                  as Lastchangedby,
      @Semantics.systemDateTime.lastChangedAt: true
      lastchangedat                  as Lastchangedat,
      @Semantics.systemDateTime.localInstanceLastChangedAt: true
      local_instance_last_changed_at as LocalInstanceLastChangedAt,
      _ITEM // Make association public
}
where
      withoutpo  <> 'X'
  and withmatdoc <> 'X'
*********************************************************************************

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Gate Pass Item View'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZI_GTPASSITEM_M
  as select from zgtpassitem
  association to parent ZI_GTPASSHEADER_M as _HEADER on $projection.Headeruuid = _HEADER.Headeruuid
{
  key itemuuid        as Itemuuid,
      headeruuid      as Headeruuid,
      slno,
//      gatepassno      as Gatepassno,
      irnnumber       as Irnnumber,
      invoiceno       as Invoiceno,
      invoicedate     as Invoicedate,
      ponumber        as Ponumber,
      irndate         as Irndate,
      ewaybillno      as Ewaybillno,
      ewaybilldate    as Ewaybilldate,
      incoterms       as Incoterms,
      shipfrmloc      as Shipfrmloc,
      odnno,
      grnno,
      @Semantics.user.createdBy: true
      created_by      as CreatedBy,
      @Semantics.systemDateTime.createdAt: true
      created_at      as CreatedAt,
      @Semantics.user.lastChangedBy: true
      last_changed_by as LastChangedBy,
      @Semantics.systemDateTime.lastChangedAt: true
      last_changed_at as LastChangedAt,
      _HEADER
}
*******************************************************************************

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'View for Inco Terms and Adrnr fields'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZI_INCOADDRESSDETAILS
  as select from I_PurchasingDocument as ekko
{
  key PurchasingDocument,
      Supplier,
      IncotermsClassification   as inco1,
      IncotermsTransferLocation as inco2,
      SupplierAddressID         as adrnr

}
******************************************************

//@AbapCatalog.preserveKey: true
//GENERATED:001:GFBfhyK17kU{lmeM6k2Sgm
//@AbapCatalog.sqlViewName: 'IPOI__VH2'
//@AbapCatalog.compiler.compareFilter: true

@VDM.viewType: #COMPOSITE

@ObjectModel.dataCategory: #VALUE_HELP
@ObjectModel.representativeKey: 'PurchaseOrderItem'

@ObjectModel.usageType.dataClass: #TRANSACTIONAL
@ObjectModel.usageType.serviceQuality: #A
@ObjectModel.usageType.sizeCategory: #L

@AccessControl.authorizationCheck: #CHECK
@AccessControl.personalData.blocking: #BLOCKED_DATA_EXCLUDED

//@ClientHandling.algorithm: #SESSION_VARIABLE

@Metadata.ignorePropagatedAnnotations: true

@EndUserText.label: 'Purchase Order Item'

define view entity ZI_PONUMBERVH
  //  as select from I_PurchasingDocument     as ekko
  as select from I_PurchasingDocumentItem as ekpo

{

      //  key ekko.PurchasingDocument

//      @ObjectModel.foreignKey.association: '_PurchasingDocument'
  key ekpo.PurchasingDocument,
      //      //  key ekpo.PurchasingDocumentItemUniqueID,
      //      ekpo.PurchasingDocumentItem,
      //      //      ekpo.PurchasingDocumentDeletionCode,
      //      //      ekpo.IsCompletelyDelivered,
      _PurchasingDocument

}
where
      ekpo.PurchasingDocumentDeletionCode <> 'L'
  and ekpo.IsCompletelyDelivered          <> 'X'
***************************************************************

//@AbapCatalog.viewEnhancementCategory: [#NONE]
//@AccessControl.authorizationCheck: #NOT_REQUIRED
//@EndUserText.label: 'Value Help For Supplier'
//@Metadata.ignorePropagatedAnnotations: true
//@Search.searchable: true
//@ObjectModel:{
//               usageType:{
//                           serviceQuality: #X,
//                           sizeCategory: #S,
//                           dataClass: #MIXED
//                         },
//                dataCategory: #VALUE_HELP,
//                representativeKey: 'Supplier',
//                supportedCapabilities: [#VALUE_HELP_PROVIDER]
//              }
//
//@Analytics.dataCategory: #DIMENSION
//@Metadata:{
//            allowExtensions: true
//          }
*********************************************************
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Supplier Value Help'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}

define view entity ZI_SUPPLERVH
  as select from I_Supplier as lfa1
{
       @ObjectModel.text.element: [ 'SupplierName' ]
       @Search.defaultSearchElement: true
       @Search.fuzzinessThreshold: 0.8
       @Search.ranking: #HIGH
  key  Supplier,
       @Semantics.text: true
       @Search.defaultSearchElement: true
       @Search.fuzzinessThreshold: 0.8
       @Search.ranking: #LOW
       SupplierName,
       PostalCode,
       CityName,
       Region
       //      @ObjectModel.text.element: [ 'SupplierName' ]
       //      @Search.defaultSearchElement: true
       //      @Search.fuzzinessThreshold: 0.8
       //  key Supplier,
       //      @Semantics.text: true
       //      @Search.defaultSearchElement: true
       //      @Search.fuzzinessThreshold: 0.8
       //      SupplierName
       //      @Semantics.text: true
       //      @Search.defaultSearchElement: true
       //      @Search.fuzzinessThreshold: 0.6
       //      concat_with_space( Supplier, SupplierName, 1 ) as SupplierCode
}
**************************************************************************

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Value Help For Vechile Capacity'
@Metadata.ignorePropagatedAnnotations: true
@Search.searchable: true
@Metadata.allowExtensions: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZI_VECHILECAPACITY
  as select from ZI_VEHICLE
{
      @UI.hidden: true
  key Id,
      @Search.defaultSearchElement: true
      VechCap,
      @Search.defaultSearchElement: true
      Vechcap_des
}
*************************************************************

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Value Help For Vechile Type'
@Metadata.ignorePropagatedAnnotations: true
@Search.searchable: true
@Metadata.allowExtensions: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
define view entity ZI_VECHILETYP
  as select from ZI_VEHICLE
{
      @UI.hidden: true
  key Id,
      @Search.defaultSearchElement: true
      VechType,
      @Search.defaultSearchElement: true
      Vechdes
}
**********************************************************

Database Tables

@EndUserText.label : 'Database Table for Gate Pass'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zgatepassheader {

  key gatepasskey   : include zgatepassheader_key not null;
  gatepassdata      : include zgatepassheader_data;
  gatepassadmindata : include zgatepassheader_admindata;

}

******************************************************************

@EndUserText.label : 'Draft table for entity ZI_GATEPASSHEADER_M'
@AbapCatalog.enhancement.category : #EXTENSIBLE_ANY
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zgatepassheaderd {

  key mandt                  : mandt not null;
  key headeruuid             : sysuuid_x16 not null;
  gatepassno                 : abap.char(20);
  gatepassdatetime           : abp_creation_tstmpl;
  vendorcode                 : lifnr;
  vendoraddress              : abap.char(100);
  lrnumber                   : abap.char(16);
  lrdate                     : abap.dats;
  plant                      : werks_d;
  vechiletype                : abap.char(10);
  vechilecapacity            : abap.char(10);
  vechilecnumber             : abap.char(16);
  transporterid              : abap.char(60);
  remarks                    : abap.char(1000);
  invremark                  : abap.char(60);
  inv_verified               : abap_boolean;
  lorryreciept               : abap.char(60);
  is_verified                : abap_boolean;
  packlist                   : abap.char(60);
  pack_verified              : abap_boolean;
  billofentry                : abap.char(60);
  boe_verified               : abap_boolean;
  other                      : abap.char(70);
  oth_verified               : abap_boolean;
  gatepassdate               : abap.dats;
  withoutpo                  : abap.char(1);
  withmatdoc                 : abap.char(1);
  createdby                  : abp_creation_user;
  createdat                  : abp_creation_tstmpl;
  lastchangedby              : abp_lastchange_user;
  lastchangedat              : abp_lastchange_tstmpl;
  localinstancelastchangedat : abp_locinst_lastchange_tstmpl;
  "%admin"                   : include sych_bdl_draft_admin_inc;

}
**********************************************************

@EndUserText.label : 'Draft table for entity ZI_GATEPASSITEM_M'
@AbapCatalog.enhancement.category : #EXTENSIBLE_ANY
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zgatepassitemd {

  key mandt     : mandt not null;
  key itemuuid  : sysuuid_x16 not null;
  headeruuid    : sysuuid_x16 not null;
  slno          : abap.char(10);
  irnnumber     : abap.char(70);
  invoiceno     : zinvno;
  invoicedate   : abap.dats;
  ponumber      : abap.char(10);
  irndate       : abap.dats;
  challanno     : zchallanno;
  @EndUserText.label : 'Challan Item No'
  challanitemno : abap.char(10);
  @EndUserText.label : 'Invoice Date'
  challandate   : abap.dats;
  @EndUserText.label : 'PO Number'
  custponumber  : abap.char(16);
  ewaybillno    : abap.char(16);
  ewaybilldate  : abap.dats;
  incoterms     : abap.char(20);
  shipfrmloc    : abap.char(25);
  odnno         : abap.char(16);
  grnno         : mblnr;
  materialdocno : mblnr;
  createdby     : abp_creation_user;
  createdat     : abp_creation_tstmpl;
  lastchangedby : abp_lastchange_user;
  lastchangedat : abp_lastchange_tstmpl;
  "%admin"      : include sych_bdl_draft_admin_inc;

}
******************************************************************************

@EndUserText.label : 'Draft table for counter value'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #ALLOWED
define table zgp_counter {

  key client                     : abap.clnt not null;
  key uuid                       : sysuuid_x16 not null;
  gpdate                         : abap.dats;
  lastnum                        : abap.numc(7);
  last_changed_by                : abp_lastchange_user;
  last_changed_at                : abp_lastchange_tstmpl;
  local_instance_last_changed_at : abp_locinst_lastchange_tstmpl;

}
****************************************************************

@EndUserText.label : 'Database Table for Gate Pass'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
@AbapCatalog.tableCategory : #TRANSPARENT
@AbapCatalog.deliveryClass : #A
@AbapCatalog.dataMaintenance : #RESTRICTED
define table zgtpassitem {

  key gtpassitemkey   : include zgtpassitem_key not null;
  gtpassitemdata      : include zgtpassitem_data;
  gtpassitemadmindata : include zgtpassitem_admin;

}
**********************************************************************
Strcutures

@EndUserText.label : 'Gate Pass Header Admin Details'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgatepassheader_admindata {

  createdby                      : abp_creation_user;
  createdat                      : abp_creation_tstmpl;
  lastchangedby                  : abp_lastchange_user;
  lastchangedat                  : abp_lastchange_tstmpl;
  local_instance_last_changed_at : abp_locinst_lastchange_tstmpl;

}
******************************************************************************

@EndUserText.label : 'Gate Pass Header Details'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgatepassheader_data {

  @EndUserText.label : 'GatePass No'
  gatepassno       : abap.char(20);
  vendorcode       : lifnr;
  @EndUserText.label : 'Vendor Address'
  vendoraddress    : abap.char(100);
  gatepassdatetime : abp_creation_tstmpl;
  @EndUserText.label : 'Lr Number'
  lrnumber         : abap.char(16);
  @EndUserText.label : 'Lr Date'
  lrdate           : abap.dats;
  plant            : werks_d not null;
  @EndUserText.label : 'Vechile Type'
  vechiletype      : abap.char(10);
  @EndUserText.label : 'Vechile Capacity'
  vechilecapacity  : abap.char(10);
  @EndUserText.label : 'Vechile Number'
  vechilecnumber   : abap.char(16);
  @EndUserText.label : 'Transporter ID'
  transporterid    : abap.char(60);
  @EndUserText.label : 'Remarks'
  remarks          : abap.char(1000);
  invremark        : abap.char(60);
  inv_verified     : abap_boolean;
  lorryreciept     : abap.char(60);
  is_verified      : abap_boolean;
  packlist         : abap.char(60);
  pack_verified    : abap_boolean;
  @EndUserText.label : 'Bill Of Entry'
  billofentry      : abap.char(60);
  boe_verified     : abap_boolean;
  other            : abap.char(70);
  oth_verified     : abap_boolean;
  @EndUserText.label : 'GatePass Date'
  gatepassdate     : abap.dats;
  @EndUserText.label : 'GatePass Without PO'
  withoutpo        : abap.char(1);
  @EndUserText.label : 'GatePass With MatDoc No'
  withmatdoc       : abap.char(1);

}
******************************************************************************

@EndUserText.label : 'Gate Pass Header Key Fields'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgatepassheader_key {

  key client     : mandt not null;
  key headeruuid : sysuuid_x16 not null;

}

**************************************************************************************

@EndUserText.label : 'Gate Pass Item Admin Data'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgtpassitem_admin {

  created_by      : abp_creation_user;
  created_at      : abp_creation_tstmpl;
  last_changed_by : abp_lastchange_user;
  last_changed_at : abp_lastchange_tstmpl;

}
***************************************************************************

@EndUserText.label : 'Gate Pass Item Data'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgtpassitem_data {

  headeruuid    : sysuuid_x16;
  @EndUserText.label : 'SL No'
  slno          : abap.char(10);
  @EndUserText.label : 'IRN Number'
  irnnumber     : abap.char(70);
  invoiceno     : zinvno;
  @EndUserText.label : 'Invoice Date'
  invoicedate   : abap.dats;
  @EndUserText.label : 'PO Number'
  ponumber      : abap.char(10);
  @EndUserText.label : 'IRN Date'
  irndate       : abap.dats;
  challanno     : zchallanno;
  @EndUserText.label : 'Challan Item No'
  challanitemno : abap.char(10);
  @EndUserText.label : 'Challan Date'
  challandate   : abap.dats;
  @EndUserText.label : 'Customer PO Number'
  custponumber  : abap.char(16);
  @EndUserText.label : 'E-Way Bill No'
  ewaybillno    : abap.char(16);
  @EndUserText.label : 'E-Way Bill Date'
  ewaybilldate  : abap.dats;
  @EndUserText.label : 'Inco Terms'
  incoterms     : abap.char(20);
  @EndUserText.label : 'Ship From Location'
  shipfrmloc    : abap.char(25);
  @EndUserText.label : 'ODN No'
  odnno         : abap.char(16);
  grnno         : mblnr;
  materialdocno : mblnr;

}
*******************************************************************

@EndUserText.label : 'Gate Pass Item Key Fields'
@AbapCatalog.enhancement.category : #NOT_EXTENSIBLE
define structure zgtpassitem_key {

  key client   : mandt not null;
  key itemuuid : sysuuid_x16 not null;

}
******************************************************************
Classes 

CLASS zbp_i_gtpassheader_m DEFINITION PUBLIC ABSTRACT FINAL FOR BEHAVIOR OF zi_gtpassheader_m.
ENDCLASS.

CLASS zbp_i_gtpassheader_m IMPLEMENTATION.
ENDCLASS.

****************************************************************

managed implementation in class zbp_i_gtpassheader_m unique;
strict ( 2 );
with draft;

define behavior for ZI_GTPASSHEADER_M alias header
persistent table zgatepassheader
draft table zgatepassheaderd
lock master
total etag LastChangedAt
authorization master ( instance )
etag master LocalInstanceLastChangedAt
{
  create; //( precheck );
  update; //( precheck );
  delete;
  association _ITEM { create ( authorization : update ); with draft; }
  field ( numbering : managed, readonly ) headeruuid;
  field ( readonly ) Gatepassno, vendoraddress, Gatepassdatetime, gatepassdate;
  field ( mandatory ) Plant, inv_verified, is_verified, Vendorcode;
  //  field ( features : instance ) oth_verified;

  draft action Resume with additional implementation;
  draft action Edit with additional implementation;
  draft action Activate optimized with additional implementation;
  draft action Discard;

  //  determination otherVerified on modify { field oth_verified; }
  determination setGatePassId on save { create; }
  determination getVendorAddress on modify { field Vendorcode; }


  validation validateplant on save { create; update; field Plant, inv_verified, is_verified; }
  side effects
  {
    field Vendorcode affects field vendoraddress;
  }

  draft determine action Prepare
  {
    validation validateplant;
    validation ZI_GTPASSITEM_M~validatefields;
    validation ZI_GTPASSITEM_M~validategatepass;
  }

  mapping for zgatepassheader corresponding
    {
      Headeruuid                 = headeruuid;
      gatepassno                 = gatepassno;
      gatepassdatetime           = gatepassdatetime;
      vendorcode                 = vendorcode;
      vendoraddress              = vendoraddress;
      lrnumber                   = lrnumber;
      lrdate                     = lrdate;
      plant                      = plant;
      vechiletype                = vechiletype;
      vechilecapacity            = vechilecapacity;
      vechilecnumber             = vechilecnumber;
      transporterid              = transporterid;
      lorryreciept               = lorryreciept;
      createdby                  = createdby;
      createdat                  = createdat;
      lastchangedby              = lastchangedby;
      lastchangedat              = lastchangedat;
      localinstancelastchangedat = local_instance_last_changed_at;
    }
}

define behavior for ZI_GTPASSITEM_M alias item
persistent table zgtpassitem
draft table zgatepassitemd
lock dependent by _HEADER
authorization dependent by _HEADER
etag master LastChangedAt
{

  update;
  delete;
  field ( numbering : managed, readonly ) Itemuuid;
  field ( readonly ) Headeruuid, Incoterms, Shipfrmloc, grnno, slno;
  //  field ( readonly : update ) PoNumber;
  field ( mandatory ) Ponumber, Invoicedate, Invoiceno;
  //  validation validatePonumber on save { create; update; field Ponumber; field Invoiceno; field Invoicedate; }
  validation validatefields on save { create; update; field Invoiceno; field Invoicedate; }
  validation validategatepass on save { create; update; field Invoiceno; field Invoicedate; }
  association _HEADER { with draft; }
  determination getserialnumber on modify { create; }
  determination getincotermsandshiploc on modify { field Ponumber; }
  //  determination touppercase on save { create; update; field Invoiceno; }
  //  validation touppercase on save { create; update; field Invoiceno; }
  side effects
  {
    field Ponumber affects field Incoterms, field Shipfrmloc, field grnno;
  }

  mapping for zgtpassitem corresponding
    {
      Itemuuid      = itemuuid;
      //      gatepassno    = gatepassno;
      irnnumber     = irnnumber;
      invoiceno     = invoiceno;
      invoicedate   = invoicedate;
      ponumber      = ponumber;
      irndate       = irndate;
      ewaybillno    = ewaybillno;
      ewaybilldate  = ewaybilldate;
      incoterms     = incoterms;
      shipfrmloc    = shipfrmloc;
      odnno         = odnno;
      grnno         = grnno;
      createdby     = created_by;
      createdat     = created_at;
      lastchangedby = last_changed_by;
      lastchangedat = last_changed_at;
    }
}
**************************************************************************************************
