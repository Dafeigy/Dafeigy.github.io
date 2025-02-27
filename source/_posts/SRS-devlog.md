---
title: Type your title here
mathjax: true
tags:
- tags
categories:
- categoreis1
---

<p align="center">
    <img src="https://s2.loli.net/2022/08/20/oxvPZeT8G7ydzct.png" alt="pyecharts logo" width=200 height=200 />
</p>


开发日志，垃圾心情记录

<!-- more -->

## 代码定位

SRS的相关实质性的信息在`openair1/SCHED_NR/phy_procedures_nr_gNB.c`文件中。快捷定位为搜索`SRS_IND_DEBUG`。目前的SRS功能仅支持codebook的case，其他的`beamManagement`、`nonCodebook`以及`antennaSwitching`没有定义。

### SRS信息

相关函数运行在`phy_procedures_gNB_uespec_RX()`中，是以一个循环执行的：

```c
  for (int i = 0; i < gNB->max_nb_srs; i++) {
    NR_gNB_SRS_t *srs = &gNB->srs[i];
    if (srs) {
      //...//
  }
```

容纳srs信息的是PHY_VARS_gNB类型的指针，其定义在`openair1/PHY/def_gNB.h`中，是一个非常庞大的结构体，和srs有关的成员有int型的`max_nb_srs`、`srs_thres`，NR_gNB_SRS_t型指针的srs、nr_srs_info_t 型的二维指针nr_srs_info以及time_stats_t型的成员`srs_channel_estimation_stats`、`srs_timing_advance_stats`、`srs_report_tlv_stats`、`srs_beam_report_stats`以及`srs_iq_matrix_stats`。

我们想提取的数据是`int32_t`类型的`srs_received_signal`,`srs_estimated_channel_freq`,`srs_estimated_channel_time`,`srs_estimated_channel_time_shifted`。这些都是在`nr_srs_channel_estimation()`函数中进行更新的。

## 2023/5/6

拿到了祖传OPPO，锁网有点问题，但是好像似乎能勉强拿到部分数据，但是运行了一会就报废了，使用的配置如下：

```
Active_gNBs = ( "gNB-OAI");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "none";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID    =  0xe00;
    gNB_name  =  "gNB-OAI";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({
                  mcc = 466;
                  mnc = 92;
                  mnc_length = 2;
                  snssaiList = (
                    {
                      sst = 1;
                    }
                  );

                  });

    nr_cellid = 12345678L;

    ////////// Physical parameters:
    # pdsch_AntennaPorts_N1                                     = 2;
    # pusch_AntennaPorts                                        = 2;
    do_SRS                                                    = 1;
    #ssb_SubcarrierOffset                                      = 0;
    # min_rxtxtime                                              = 6;
    # sib1_tda                                                  = 4;


     pdcch_ConfigSIB1 = (
      {
        controlResourceSetZero = 12;
        searchSpaceZero = 10;
      }
      );

    servingCellConfigCommon = (
    {
 #spCellConfigCommon

      physCellId                                                    = 0;

#  downlinkConfigCommon
    #frequencyInfoDL
      # this is 3600 MHz + 43 PRBs@30kHz SCS (same as initial BWP)
      absoluteFrequencySSB                                             = 641280;
      dl_frequencyBand                                                 = 78;
      # this is 3600 MHz
      dl_absoluteFrequencyPointA                                       = 640008;
      #scs-SpecificCarrierList
        dl_offstToCarrier                                              = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        dl_subcarrierSpacing                                           = 1;
        dl_carrierBandwidth                                            = 106;
     #initialDownlinkBWP
      #genericParameters
        # this is RBstart=27,L=48 (275*(L-1))+RBstart
        initialDLBWPlocationAndBandwidth                               = 28875; # 6366 12925 12956 28875 12952
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialDLBWPsubcarrierSpacing                                   = 1;
      #pdcch-ConfigCommon
        initialDLBWPcontrolResourceSetZero                              = 12;
        initialDLBWPsearchSpaceZero                                     = 10;

  #uplinkConfigCommon
     #frequencyInfoUL
      ul_frequencyBand                                              = 78;
      #scs-SpecificCarrierList
      ul_offstToCarrier                                             = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      ul_subcarrierSpacing                                          = 1;
      ul_carrierBandwidth                                           = 106;
      pMax                                                          = 20;
     #initialUplinkBWP
      #genericParameters
        initialULBWPlocationAndBandwidth                            = 28875;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialULBWPsubcarrierSpacing                               = 1;
      #rach-ConfigCommon
        #rach-ConfigGeneric
          prach_ConfigurationIndex                                  = 98;
#prach_msg1_FDM
#0 = one, 1=two, 2=four, 3=eight
          prach_msg1_FDM                                            = 0;
          prach_msg1_FrequencyStart                                 = 0;
          zeroCorrelationZoneConfig                                 = 13;
          preambleReceivedTargetPower                               = -96;
#preamblTransMax (0...10) = (3,4,5,6,7,8,10,20,50,100,200)
          preambleTransMax                                          = 9;
#powerRampingStep
# 0=dB0,1=dB2,2=dB4,3=dB6
        powerRampingStep                                            = 1;
#ra_ReponseWindow
#1,2,4,8,10,20,40,80
        ra_ResponseWindow                                           = 4;
#ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR
#1=oneeighth,2=onefourth,3=half,4=one,5=two,6=four,7=eight,8=sixteen
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR                = 4;
#oneHalf (0..15) 4,8,12,16,...60,64
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB                   = 14;
#ra_ContentionResolutionTimer
#(0..7) 8,16,24,32,40,48,56,64
        ra_ContentionResolutionTimer                                = 7;
        rsrp_ThresholdSSB                                           = 19;
#prach-RootSequenceIndex_PR
#1 = 839, 2 = 139
        prach_RootSequenceIndex_PR                                  = 2;
        prach_RootSequenceIndex                                     = 1;
        # SCS for msg1, can only be 15 for 30 kHz < 6 GHz, takes precendence over the one derived from prach-ConfigIndex
        #
        msg1_SubcarrierSpacing                                      = 1,
# restrictedSetConfig
# 0=unrestricted, 1=restricted type A, 2=restricted type B
        restrictedSetConfig                                         = 0,

        msg3_DeltaPreamble                                          = 1;
        p0_NominalWithGrant                                         =-80;

# pucch-ConfigCommon setup :
# pucchGroupHopping
# 0 = neither, 1= group hopping, 2=sequence hopping
        pucchGroupHopping                                           = 0;
        hoppingId                                                   = 40;
        p0_nominal                                                  = -80;
# ssb_PositionsInBurs_BitmapPR
# 1=short, 2=medium, 3=long
      ssb_PositionsInBurst_PR                                       = 2;
      ssb_PositionsInBurst_Bitmap                                   = 1;

# ssb_periodicityServingCell
# 0 = ms5, 1=ms10, 2=ms20, 3=ms40, 4=ms80, 5=ms160, 6=spare2, 7=spare1
      ssb_periodicityServingCell                                    = 2;

# dmrs_TypeA_position
# 0 = pos2, 1 = pos3
      dmrs_TypeA_Position                                           = 0;

# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      subcarrierSpacing                                             = 1;


  #tdd-UL-DL-ConfigurationCommon
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      referenceSubcarrierSpacing                                    = 1;
      # pattern1
      # dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                 = 6;
      nrofDownlinkSlots                                             = 7;
      nrofDownlinkSymbols                                           = 6;
      nrofUplinkSlots                                               = 2;
      nrofUplinkSymbols                                             = 4;

      ssPBCH_BlockPower                                             = -25;
  }

  );


    # ------- SCTP definitions
    SCTP :
    {
        # Number of streams to use in input/output
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };


    ////////// AMF parameters:
 
    ////////// MME parameters:
    amf_ip_address      = ( { ipv4       = "202.204.62.158";
                              ipv6       = "192:168:30::17";
                              active     = "yes";
                              preference = "ipv4";
                            }
                          );


    NETWORK_INTERFACES :
    {
        GNB_INTERFACE_NAME_FOR_NG_AMF            = "eno1";
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.1.248/24";
        GNB_INTERFACE_NAME_FOR_NGU               = "eno1";
        GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.1.248/24";
        GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
    };

  }
);

MACRLCs = (
{
  num_cc                      = 1;
  tr_s_preference             = "local_L1";
  tr_n_preference             = "local_RRC";
  pusch_TargetSNRx10          = 200;
  pucch_TargetSNRx10          = 200;
  ulsch_max_frame_inactivity  = 0;
}
);

L1s = (
{
  num_cc = 1;
  tr_n_preference       = "local_mac";
	pusch_proc_threads = 8;
  prach_dtx_threshold   = 120;
  pucch0_dtx_threshold  = 150;
  ofdm_offset_divisor   = 8; #set this to UINT_MAX for offset 0
}
);

RUs = (
{
  local_rf       = "yes"
  nb_tx          = 1
  nb_rx          = 1
  att_tx         = 6
  att_rx         = 6;
  bands          = [78];
  max_pdschReferenceSignalPower = -27;
  max_rxgain                    = 114;
  sf_extension                  = 0;
  eNB_instances  = [0];
  #beamforming 1x4 matrix:
  bf_weights = [0x00007fff, 0x0000, 0x0000, 0x0000];
  clock_src = "internal";
  # sdr_addrs = "use_dpdk=1,type=n3xx,addr=192.168.10.2,mgmt_addr=192.168.1.124"
}
);

THREAD_STRUCT = (
{
  #three config for level of parallelism "PARALLEL_SINGLE_THREAD", "PARALLEL_RU_L1_SPLIT", or "PARALLEL_RU_L1_TRX_SPLIT"
  parallel_config    = "PARALLEL_SINGLE_THREAD";
  #two option for worker "WORKER_DISABLE" or "WORKER_ENABLE"
  worker_config      = "WORKER_ENABLE";
}
);

rfsimulator :
{
  serveraddr = "server";
  serverport = "4043";
  options = (); #("saviq"); or/and "chanmod"
  modelname = "AWGN";
  IQfile = "/tmp/rfsimulator.iqs";
};

security = {
  # preferred ciphering algorithms
  # the first one of the list that an UE supports in chosen
  # valid values: nea0, nea1, nea2, nea3
  ciphering_algorithms = ( "nea0" );

  # preferred integrity algorithms
  # the first one of the list that an UE supports in chosen
  # valid values: nia0, nia1, nia2, nia3
  integrity_algorithms = ( "nia2", "nia0" );

  # setting 'drb_ciphering' to "no" disables ciphering for DRBs, no matter
  # what 'ciphering_algorithms' configures; same thing for 'drb_integrity'
  drb_ciphering = "yes";
  drb_integrity = "no";
};

log_config :
{
  global_log_level                      ="info";
  hw_log_level                          ="error";
  phy_log_level                         ="info";
  mac_log_level                         ="info";
  rlc_log_level                         ="info";
  pdcp_log_level                        ="info";
  rrc_log_level                         ="info";
  ngap_log_level                        ="debug";
  f1ap_log_level                        ="debug";
};
```

输出Log如下：

```
[INFO] [UHD] linux; GNU C++ version 9.4.0; Boost_107100; UHD_4.4.0.HEAD-0-g5fac246b
[INFO] [B200] Detected Device: B210
[INFO] [B200] Operating over USB 3.
[INFO] [B200] Initialize CODEC control...
[INFO] [B200] Initialize Radio control...
[INFO] [B200] Performing register loopback test... 
[INFO] [B200] Register loopback test passed
[INFO] [B200] Performing register loopback test... 
[INFO] [B200] Register loopback test passed
[INFO] [B200] Asking for clock rate 30.720000 MHz... 
[INFO] [B200] Actually got clock rate 30.720000 MHz.
-- Using calibration table: calib_table_b210_38
[INFO] [B200] Asking for clock rate 46.080000 MHz... 
[INFO] [B200] Actually got clock rate 46.080000 MHz.
Using Device: Single USRP:
  Device: B-Series Device
  Mboard 0: B210
  RX Channel: 0
    RX DSP: 0
    RX Dboard: A
    RX Subdev: FE-RX2
  RX Channel: 1
    RX DSP: 1
    RX Dboard: A
    RX Subdev: FE-RX1
  TX Channel: 0
    TX DSP: 0
    TX Dboard: A
    TX Subdev: FE-TX2
  TX Channel: 1
    TX DSP: 1
    TX Dboard: A
    TX Subdev: FE-TX1

setup_RU_buffers: frame_parms = 0x7fd5d0563010
[PHY]   RU 0 Setting N_TA_offset to 600 samples (factor 1.500000, UL Freq 3600120, N_RB 106, mu 1)
[PHY]   Signaling main thread that RU 0 is ready, sl_ahead 6
waiting for sync (ru_thread,-1/0x556463a5b2d4,0x5564653ed8a0,0x5564653ecae0)
RC.ru_mask:00
[PHY]   RUs configured
ALL RUs READY!
RC.nb_RU:1
ALL RUs ready - init gNBs
Not NFAPI mode - call init_eNB_afterRU()
[PHY]   init_eNB_afterRU() RC.nb_nr_inst:1
[PHY]   RC.nb_nr_CC[inst:0]:0x7fd5d1eb7010
[PHY]   [gNB 0] phy_init_nr_gNB() About to wait for gNB to be configured
[LIBCONFIG] loader.dfts: 2/2 parameters successfully set, (1 to default value)
shlib_path libdfts.so
[LOADER] library libdfts.so successfully loaded
[LIBCONFIG] loader.ldpc: 2/2 parameters successfully set, (1 to default value)
shlib_path libldpc.so
[LOADER] library libldpc.so successfully loaded
[PHY]   Initialise nr transport
[PHY]   Mapping RX ports from 1 RUs to gNB 0
[PHY]   gNB->num_RU:1
[PHY]   Attaching RU 0 antenna 0 to gNB antenna 0
create a thread for core -1
create a thread for core -1
create a thread for core -1
create a thread for core -1
create a thread for core -1
create a thread for core -1
create a thread for core -1
create a thread for core -1
[PHY]   Creating thread for TX reordering and dispatching to RU
waiting for sync (L1_stats_thread,-1/0x556463a5b2d4,0x5564653ed8a0,0x5564653ecae0)
ALL RUs ready - ALL gNBs ready
Sending sync to all threads
Entering ITTI signals handler
TYPE <CTRL-C> TO TERMINATE
got sync (L1_stats_thread)
got sync (ru_thread)
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started opp_enabled 0
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
[PHY]   tx_reorder_thread started
[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_PHY]   [gNB 0][RAPROC] Frame 629, slot 19 Initiating RA procedure with preamble 11, energy 51.0 dB (I0 122, thres 120), delay 4 start symbol 0 freq index 0
[MAC]   UL_info[Frame 629, Slot 19] Calling initiate_ra_proc RACH:SFN/SLOT:629/19
[NR_MAC]   Search for not existing rnti (ignore for RA): e3f8
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 629 Activating Msg2 generation in frame 630, slot 7 using RA rnti 10b SSB, new rnti e3f8 index 0 RA index 0
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 630, slotP 7: Generating RA-Msg2 DCI, rnti 0x10b, state 1, CoreSetType 2
[NR_MAC]   [RAPROC] Msg3 slot 17: current slot 7 Msg3 frame 630 k2 7 Msg3_tda_id 3
[NR_MAC]   [gNB 0][RAPROC] Frame 630, Subframe 7: rnti e3f8 RA state 2
[NR_MAC]   Search for not existing rnti (ignore for RA): e3f8
[NR_MAC]   Adding UE with rnti 0xe3f8
[NR_MAC]   [gNB 0][RAPROC] PUSCH with TC_RNTI 0xe3f8 received correctly, adding UE MAC Context RNTI 0xe3f8
[NR_MAC]   [RAPROC] RA-Msg3 received (sdu_lenP 7)
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE e3f8
[NR_MAC]   Added new CBRA process for UE RNTI e3f8 with initial CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[RLC]   activated srb0 for UE with RNTI 0xe3f8
[RLC]   ../../../openair2/LAYER2/nr_rlc/nr_rlc_oai_api.c:693:nr_rlc_add_srb: added srb 1 to UE with RNTI 0xe3f8
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE e3f8
[NR_MAC]   Activating scheduling RA-Msg4 for TC_RNTI 0xe3f8 (state 2)
[NR_MAC]   Unexpected ULSCH HARQ PID 0 (have -1) for RNTI 0xe3f8 (ignore this warning for RA)
[RRC]   initial UL RRC message nr_cellid 0 does not match RRC's 12345678
[NR_RRC]   Decoding CCCH: RNTI e3f8, payload_size 6
[NR_RRC]   search by rnti not found e3f8
[NR_RRC]   search by rnti not found e3f8
[NR_RRC]   Returning new RRC UE context RRC ue id: 0
[NR_RRC]    Created new UE context rnti: e3f8, random ue id e2a3ba6c0a000000, RRC ue id 0
[NR_RRC]   rrc_gNB_generate_RRCSetup for RNTI e3f8
[NR_MAC]   DL RRC Message Transfer with 137 bytes for RNTI e3f8 SRB 0
[NR_MAC]   ( 631. 5) SRB0 has 137 bytes
[NR_MAC]   Generate msg4, rnti: e3f8
[NR_MAC]   Encoded RRCSetup Piggyback (137 + 2 bytes), mac_pdu_length 146
[NR_MAC]   (UE RNTI 0xe3f8) Received Ack of RA-Msg4. CBRA procedure succeeded!
[NR_MAC]   (631.13) Activating RRC processing timer for UE e3f8 with 10 ms
[NR_MAC]   (632.13) De-activating RRC processing timer for UE e3f8
[NR_MAC]   Modified rnti e3f8 with CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] [RAPROC] Logical Channel UL-DCCH, processing NR_RRCSetupComplete from UE (SRB1 Active)
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] UE State = NR_RRC_CONNECTED 
[NGAP]   [gNB 0] Chose AMF 'ngc-amf' (assoc_id 114) through selected PLMN Identity index 0 MCC 466 MNC 92
[NR_MAC]   DL RRC Message Transfer with 51 bytes for RNTI e3f8 SRB 1
[NR_MAC]   Frame.Slot 640.0
UE RNTI e3f8 (1) PH 49 dB PCMAX 21 dBm, average RSRP 0 (0 meas)
UE e3f8: CQI 0, RI 1, PMI (0,0)
UE e3f8: dlsch_rounds 3/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS 9
UE e3f8: dlsch_total_bytes 246
UE e3f8: ulsch_rounds 25/0/0/0, ulsch_DTX 0, ulsch_errors 0, BLER 0.10000 MCS 9
UE e3f8: ulsch_total_bytes_scheduled 2072, ulsch_total_bytes_received 1840
UE e3f8: LCID 1: 56 bytes TX

[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NR_MAC]   DL RRC Message Transfer with 28 bytes for RNTI e3f8 SRB 1
[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 110)
[NGAP]   could not found NGAP_ProtocolIE_ID_id_UEAggregateMaximumBitRate
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 71)
[NGAP]   AllowedNSSAI.list.count 1
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 36)
[RRC]   selecting ciphering algorithm 0
[RRC]   selecting integrity algorithm 2
[NR_RRC]   [gNB 0][UE e3f8] Selected security algorithms (0x7fd5c4004c84): 0, 2, changed
[NR_RRC]   [gNB 0][UE e3f8] Saved security key 8BE43001662E75E97CA39F38510C56EC0D2CE0AEC754A8EA968ADD884A249849
[NR_RRC]   
KgNB:8b e4 30 01 66 2e 75 e9 7c a3 9f 38 51 0c 56 ec 0d 2c e0 ae c7 54 a8 ea 96 8a dd 88 4a 24 98 49 
[NR_RRC]   
KRRCenc:6e 8b 94 a6 3c 7c 44 2d a8 9c e5 71 7f 4a 5c 73 
[NR_RRC]   
KRRCint:a7 a0 e3 f0 85 b1 3f ba 85 5d b3 32 27 2c 5e 04 
[NR_RRC]   UE e3f8 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[NR_MAC]   DL RRC Message Transfer with 9 bytes for RNTI e3f8 SRB 1
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] received securityModeComplete on UL-DCCH 1 from UE
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] Logical Channel DL-DCCH, Generate NR UECapabilityEnquiry (bytes 8)
[NR_MAC]   DL RRC Message Transfer with 14 bytes for RNTI e3f8 SRB 1
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] received ueCapabilityInformation on UL-DCCH 1 from UE
[NR_RRC]   got UE capabilities for UE e3f8
[NR_RRC]   Send message to ngap: NGAP_UE_CAPABILITIES_IND
[NR_RRC]   SRS configured with 1 ports
[NR_RRC]   do_RRCReconfiguration(): size 224
[NR_RRC]   [gNB 0] Frame 0, Logical Channel DL-DCCH, Generate NR_RRCReconfiguration (bytes 224, UE id e3f8)
[NR_MAC]   DL RRC Message Transfer with 230 bytes for RNTI e3f8 SRB 1
[NR_MAC]   Added new CBRA process for UE RNTI e3f8 with initial CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_MAC]   Activating RRC processing timer for UE e3f8 with 10 ms
[NR_MAC]   (672.19) De-activating RRC processing timer for UE e3f8
[NR_MAC]   Modified rnti e3f8 with CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_RRC]   Receive RRC Reconfiguration Complete message UE e3f8
[PDCP]   nr_pdcp_add_srbs() with void list
[PDCP]   nr_pdcp_add_drbs() with void list
[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NR_RRC]   Send message to sctp: NGAP_InitialContextSetupResponse
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 680
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real 76, imag -40
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real 68, imag -60
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real 48, imag -68
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real 56, imag -64
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real 52, imag -80
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real 40, imag -84
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real 28, imag -88
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real 20, imag -100
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real 8, imag -96
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real 0, imag -96
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real -8, imag -92
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real -24, imag -108
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real -32, imag -100
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real -52, imag -92
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real -56, imag -88
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real -60, imag -80
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real -72, imag -84
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real -76, imag -76
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real -92, imag -64
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real -80, imag -48
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real -92, imag -40
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real -104, imag -24
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real -104, imag -24
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real -104, imag -12
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real -96, imag 4
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real -104, imag 8
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real -100, imag 20
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real -96, imag 24
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real -96, imag 48
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real -76, imag 56
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real -72, imag 60
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real -56, imag 64
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real -56, imag 68
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real -56, imag 72
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real -44, imag 76
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real -28, imag 76
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real -24, imag 72
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real -12, imag 80
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real -4, imag 84
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real 4, imag 68
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real 12, imag 80
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real 12, imag 68
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real 24, imag 64
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real 36, imag 52
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real 44, imag 52
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real 44, imag 44
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real 48, imag 36
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real 52, imag 36
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real 52, imag 24
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 60, imag 8
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 56, imag 16
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real 64, imag 4
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 72, imag -8
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 76, imag -12
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real 60, imag -20
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real 52, imag -36
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real 56, imag -24
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real 32, imag -28
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real 44, imag -40
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real 32, imag -48
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real 28, imag -48
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real 24, imag -40
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real 12, imag -40
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real 16, imag -40
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real 12, imag -28
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real 4, imag -36
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real -4, imag -32
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real 0, imag -36
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real -8, imag -36
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real 0, imag -28
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real 0, imag -40
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real -8, imag -24
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real -8, imag -32
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real -12, imag -32
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real 0, imag -28
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real 4, imag -28
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real -12, imag -24
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real -24, imag -36
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real -16, imag -24
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -8, imag -28
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real -12, imag -24
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real -16, imag -24
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real -20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real -16, imag -24
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real -12, imag -24
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real -20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real -20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real -20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real -28, imag -24
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real -28, imag -28
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real -32, imag -32
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real -32, imag -28
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real -36, imag -16
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real -28, imag -20
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real -40, imag -20
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real -28, imag -20
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real -28, imag -8
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -36, imag -8
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real -32, imag -4
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real -36, imag 0
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real -28, imag 4
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real -40, imag 4
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real -32, imag 12
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real -36, imag 12
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 688
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real 32, imag -12
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real 36, imag -24
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real 20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real 24, imag -32
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real 16, imag -40
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real 12, imag -48
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real 16, imag -36
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real 28, imag -44
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real 8, imag -36
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real 4, imag -44
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real 4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real -4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real -20, imag -56
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real -16, imag -52
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real -8, imag -44
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real -20, imag -40
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real -28, imag -44
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real -24, imag -52
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real -32, imag -44
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real -36, imag -44
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real -32, imag -40
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real -40, imag -36
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real -44, imag -28
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real -40, imag -28
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real -36, imag -24
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real -44, imag -20
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real -44, imag -16
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real -40, imag -16
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real -40, imag -4
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real -48, imag -8
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real -48, imag -4
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real -36, imag 8
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real -36, imag 4
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real -44, imag 12
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real -32, imag 12
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real -44, imag 16
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real -36, imag 24
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real -28, imag 16
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real -32, imag 24
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real -40, imag 20
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real -20, imag 20
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real -20, imag 28
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real -12, imag 32
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real -16, imag 28
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real -8, imag 28
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real -4, imag 20
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real -4, imag 24
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real 16, imag 28
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real 4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 12, imag 16
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 16, imag 28
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real 8, imag 12
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real 4, imag 12
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real 24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real 20, imag 4
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real 16, imag 8
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real 20, imag 0
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real 24, imag 8
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real 20, imag -12
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real 12, imag -4
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real 12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real 12, imag -4
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real -4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real 0, imag -20
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real 4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real 4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real 16, imag -8
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real -4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real 0, imag -24
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real -4, imag -28
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real -8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real -16, imag -28
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real -16, imag -20
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real -12, imag -24
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real -20, imag -20
[NR_MAC]   DL RRC Message Transfer with 43 bytes for RNTI e3f8 SRB 1
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 696
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real 12, imag -44
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real -4, imag -44
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real -8, imag -44
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real -4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real -12, imag -40
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real -12, imag -44
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real -16, imag -48
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real -24, imag -36
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real -24, imag -44
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real -16, imag -48
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real -32, imag -40
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real -36, imag -28
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real -28, imag -36
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real -36, imag -36
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real -48, imag -28
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real -44, imag -20
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real -48, imag -20
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real -44, imag -24
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real -48, imag -16
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real -52, imag -12
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real -48, imag 0
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real -64, imag -4
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real -44, imag 0
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real -44, imag 4
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real -52, imag 4
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real -44, imag 16
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real -44, imag 4
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real -44, imag 8
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real -40, imag 20
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real -40, imag 24
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real -28, imag 36
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real -36, imag 24
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real -24, imag 32
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real -24, imag 24
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real -24, imag 28
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real -16, imag 24
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real -16, imag 32
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real -16, imag 28
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real -12, imag 40
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real -8, imag 32
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real -4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real -4, imag 32
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real 4, imag 36
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real 12, imag 28
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real 12, imag 32
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real 4, imag 24
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real 12, imag 24
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real 12, imag 24
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real 12, imag 12
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 12, imag 12
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 24, imag 12
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real 20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 16, imag 4
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 12, imag 0
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real 20, imag 0
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real 16, imag -4
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real 4, imag 0
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real 20, imag -4
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real 12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real 12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real 0, imag -24
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real 4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real -4, imag 0
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real -8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real 4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real -12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real 0, imag -20
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real 4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real 4, imag -28
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real -4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real 0, imag -28
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real 0, imag -24
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real 0, imag -20
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real -12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real -4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real -16, imag -20
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real -8, imag -24
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 704
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real -16, imag -32
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real -20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real -24, imag -36
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real -20, imag -32
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real -24, imag -36
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real -36, imag -32
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real -36, imag -28
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real -32, imag -24
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real -36, imag -36
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real -36, imag -20
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real -36, imag -20
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real -48, imag -16
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real -44, imag -12
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real -36, imag -16
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real -44, imag -12
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real -40, imag -4
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real -44, imag -12
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real -44, imag 0
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real -44, imag -4
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real -40, imag 8
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real -40, imag 4
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real -40, imag 16
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real -32, imag 16
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real -40, imag 4
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real -36, imag 20
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real -36, imag 16
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real -28, imag 16
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real -32, imag 28
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real -28, imag 40
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real -28, imag 28
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real -24, imag 32
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real -20, imag 24
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real -12, imag 28
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real -20, imag 28
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real -4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real 0, imag 24
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real -4, imag 32
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real -8, imag 28
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real 4, imag 20
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real -4, imag 32
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real 0, imag 28
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real 16, imag 16
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real 4, imag 8
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real 4, imag 12
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real 16, imag 16
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real 20, imag 16
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real 16, imag 16
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real 20, imag 24
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real 16, imag 8
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 12, imag 0
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 20, imag 8
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 48, imag -20
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real 12, imag -4
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real 24, imag -12
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real 12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real 16, imag -8
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real 16, imag -4
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real 28, imag -4
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real 12, imag -4
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real 16, imag -12
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real 12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real 16, imag -12
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real 8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real -4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real -4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -4, imag 0
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real 4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real -4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real -4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real -8, imag -24
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real -4, imag -24
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real -12, imag -28
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 712
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real 24, imag -20
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real 28, imag -40
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real 32, imag -28
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real 28, imag -28
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real 20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real 28, imag -32
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real 20, imag -36
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real 20, imag -40
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real 16, imag -40
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real 16, imag -40
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real 12, imag -36
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real 8, imag -44
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real 20, imag -44
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real 4, imag -32
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real 4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real -8, imag -40
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real 0, imag -48
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real -4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real -4, imag -48
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real -8, imag -44
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real -12, imag -52
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real -12, imag -44
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real -20, imag -44
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real -24, imag -44
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real -20, imag -44
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real -32, imag -44
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real -32, imag -32
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real -28, imag -40
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real -32, imag -32
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real -28, imag -36
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real -32, imag -32
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real -28, imag -40
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real -36, imag -28
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real -40, imag -28
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real -28, imag -24
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real -44, imag -36
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real -32, imag -16
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real -36, imag -12
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real -36, imag -20
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real -40, imag -8
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real -40, imag -16
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real -40, imag -4
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real -32, imag -8
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real -40, imag 4
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real -28, imag -4
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real -44, imag 0
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real -32, imag -4
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real -40, imag 4
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real -36, imag -4
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real -32, imag 8
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real -36, imag 0
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real -28, imag 8
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real -20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real -24, imag 8
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real -16, imag 8
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real -32, imag 12
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real -24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real -12, imag 16
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real -20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real -16, imag 12
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real -20, imag 16
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real -24, imag 8
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real -16, imag 4
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real -8, imag 8
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real -8, imag 0
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real -4, imag 0
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real -8, imag 8
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real -12, imag 4
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real -4, imag 4
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real -16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real -16, imag 8
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real -8, imag 0
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real -12, imag 0
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real -16, imag -4
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real -8, imag 0
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real -8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real -8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real -4, imag 4
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real -12, imag 16
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real -16, imag 0
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real -4, imag 4
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real -20, imag 8
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real -12, imag 8
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real -8, imag 8
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real -8, imag 4
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real -8, imag 8
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real -4, imag 12
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real -8, imag 12
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real -4, imag 12
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -4, imag 4
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real -4, imag 8
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real 0, imag 8
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real 4, imag 12
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real 0, imag 8
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real 0, imag 8
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real 0, imag 4
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 720
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real 0, imag 28
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real 0, imag 24
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real 8, imag 20
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real 8, imag 24
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real 8, imag 28
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real 8, imag 24
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real 16, imag 16
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real 20, imag 24
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real 12, imag 24
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real 24, imag 20
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real 12, imag 12
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real 20, imag 20
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real 24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real 24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real 20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real 20, imag 8
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real 36, imag 8
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real 32, imag -4
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real 32, imag 8
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real 28, imag -4
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real 28, imag -8
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real 36, imag 4
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real 32, imag -8
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real 32, imag -12
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real 40, imag -8
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real 24, imag -8
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real 28, imag -16
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real 32, imag -8
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real 28, imag -12
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real 20, imag -20
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real 28, imag -20
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real 20, imag -24
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real 28, imag -24
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real 20, imag -16
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real 20, imag -24
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real 24, imag -28
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real 20, imag -24
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real 16, imag -28
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real 20, imag -40
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real 20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real 8, imag -28
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real 20, imag -28
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real 4, imag -36
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real 4, imag -40
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real 0, imag -32
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real 0, imag -32
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real -4, imag -28
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real -4, imag -32
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 0, imag -28
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 4, imag -24
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real -8, imag -36
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 0, imag -36
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real -12, imag -28
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real -8, imag -28
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real -16, imag -28
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real -8, imag -24
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real -16, imag -24
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real -16, imag -20
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real -12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real -16, imag -16
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real -16, imag -16
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real -12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real -8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real -12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real -20, imag -16
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real -12, imag -4
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real -12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real -16, imag -8
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real -8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real -8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real -12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real -4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real -12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real -12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real -8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real -12, imag -24
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real -12, imag -16
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real -8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real -16, imag -16
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real -12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real -12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real -12, imag -12
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real -16, imag -12
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real -16, imag -12
[NR_PHY]   gNB->UL_INFO.srs_ind.sfn = 728
[NR_PHY]   gNB->UL_INFO.srs_ind.slot = 8
[NR_PHY]   srs_indication->rnti = e3f8
[NR_PHY]   srs_indication->timing_advance = 31
[NR_PHY]   srs_indication->timing_advance_offset_nsec = 0
[NR_PHY]   srs_indication->srs_usage = 1
[NR_PHY]   srs_indication->report_type = 1
[NR_PHY]   nr_srs_channel_iq_matrix.normalized_iq_representation = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_gnb_antenna_elements = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_ue_srs_ports = 1
[NR_PHY]   nr_srs_channel_iq_matrix.prg_size = 1
[NR_PHY]   nr_srs_channel_iq_matrix.num_prgs = 104
[NR_PHY]   (uI 0, gI 0, pI 0) channel_matrix --> real -24, imag 24
[NR_PHY]   (uI 0, gI 0, pI 1) channel_matrix --> real -16, imag 28
[NR_PHY]   (uI 0, gI 0, pI 2) channel_matrix --> real -12, imag 16
[NR_PHY]   (uI 0, gI 0, pI 3) channel_matrix --> real -8, imag 20
[NR_PHY]   (uI 0, gI 0, pI 4) channel_matrix --> real -24, imag 20
[NR_PHY]   (uI 0, gI 0, pI 5) channel_matrix --> real -12, imag 24
[NR_PHY]   (uI 0, gI 0, pI 6) channel_matrix --> real -8, imag 28
[NR_PHY]   (uI 0, gI 0, pI 7) channel_matrix --> real -12, imag 32
[NR_PHY]   (uI 0, gI 0, pI 8) channel_matrix --> real -8, imag 24
[NR_PHY]   (uI 0, gI 0, pI 9) channel_matrix --> real 0, imag 28
[NR_PHY]   (uI 0, gI 0, pI 10) channel_matrix --> real -4, imag 24
[NR_PHY]   (uI 0, gI 0, pI 11) channel_matrix --> real -8, imag 36
[NR_PHY]   (uI 0, gI 0, pI 12) channel_matrix --> real -4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 13) channel_matrix --> real 4, imag 32
[NR_PHY]   (uI 0, gI 0, pI 14) channel_matrix --> real 4, imag 24
[NR_PHY]   (uI 0, gI 0, pI 15) channel_matrix --> real 12, imag 24
[NR_PHY]   (uI 0, gI 0, pI 16) channel_matrix --> real 4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 17) channel_matrix --> real 8, imag 24
[NR_PHY]   (uI 0, gI 0, pI 18) channel_matrix --> real 8, imag 28
[NR_PHY]   (uI 0, gI 0, pI 19) channel_matrix --> real 8, imag 16
[NR_PHY]   (uI 0, gI 0, pI 20) channel_matrix --> real 8, imag 20
[NR_PHY]   (uI 0, gI 0, pI 21) channel_matrix --> real 4, imag 28
[NR_PHY]   (uI 0, gI 0, pI 22) channel_matrix --> real 20, imag 20
[NR_PHY]   (uI 0, gI 0, pI 23) channel_matrix --> real 24, imag 20
[NR_PHY]   (uI 0, gI 0, pI 24) channel_matrix --> real 20, imag 20
[NR_PHY]   (uI 0, gI 0, pI 25) channel_matrix --> real 8, imag 24
[NR_PHY]   (uI 0, gI 0, pI 26) channel_matrix --> real 16, imag 24
[NR_PHY]   (uI 0, gI 0, pI 27) channel_matrix --> real 12, imag 16
[NR_PHY]   (uI 0, gI 0, pI 28) channel_matrix --> real 24, imag 16
[NR_PHY]   (uI 0, gI 0, pI 29) channel_matrix --> real 20, imag 16
[NR_PHY]   (uI 0, gI 0, pI 30) channel_matrix --> real 20, imag 12
[NR_PHY]   (uI 0, gI 0, pI 31) channel_matrix --> real 28, imag 8
[NR_PHY]   (uI 0, gI 0, pI 32) channel_matrix --> real 28, imag 4
[NR_PHY]   (uI 0, gI 0, pI 33) channel_matrix --> real 24, imag 8
[NR_PHY]   (uI 0, gI 0, pI 34) channel_matrix --> real 20, imag 4
[NR_PHY]   (uI 0, gI 0, pI 35) channel_matrix --> real 24, imag 8
[NR_PHY]   (uI 0, gI 0, pI 36) channel_matrix --> real 16, imag 12
[NR_PHY]   (uI 0, gI 0, pI 37) channel_matrix --> real 32, imag 4
[NR_PHY]   (uI 0, gI 0, pI 38) channel_matrix --> real 20, imag -8
[NR_PHY]   (uI 0, gI 0, pI 39) channel_matrix --> real 28, imag 0
[NR_PHY]   (uI 0, gI 0, pI 40) channel_matrix --> real 24, imag -4
[NR_PHY]   (uI 0, gI 0, pI 41) channel_matrix --> real 20, imag -4
[NR_PHY]   (uI 0, gI 0, pI 42) channel_matrix --> real 24, imag 0
[NR_PHY]   (uI 0, gI 0, pI 43) channel_matrix --> real 20, imag -4
[NR_PHY]   (uI 0, gI 0, pI 44) channel_matrix --> real 20, imag -8
[NR_PHY]   (uI 0, gI 0, pI 45) channel_matrix --> real 20, imag -8
[NR_PHY]   (uI 0, gI 0, pI 46) channel_matrix --> real 20, imag -8
[NR_PHY]   (uI 0, gI 0, pI 47) channel_matrix --> real 28, imag -16
[NR_PHY]   (uI 0, gI 0, pI 48) channel_matrix --> real 16, imag -8
[NR_PHY]   (uI 0, gI 0, pI 49) channel_matrix --> real 20, imag -8
[NR_PHY]   (uI 0, gI 0, pI 50) channel_matrix --> real 8, imag -20
[NR_PHY]   (uI 0, gI 0, pI 51) channel_matrix --> real 20, imag -12
[NR_PHY]   (uI 0, gI 0, pI 52) channel_matrix --> real 12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 53) channel_matrix --> real 20, imag -16
[NR_PHY]   (uI 0, gI 0, pI 54) channel_matrix --> real 8, imag -24
[NR_PHY]   (uI 0, gI 0, pI 55) channel_matrix --> real 16, imag -20
[NR_PHY]   (uI 0, gI 0, pI 56) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 57) channel_matrix --> real 16, imag -8
[NR_PHY]   (uI 0, gI 0, pI 58) channel_matrix --> real 12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 59) channel_matrix --> real 16, imag -20
[NR_PHY]   (uI 0, gI 0, pI 60) channel_matrix --> real 16, imag -16
[NR_PHY]   (uI 0, gI 0, pI 61) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 62) channel_matrix --> real 0, imag -24
[NR_PHY]   (uI 0, gI 0, pI 63) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 64) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 65) channel_matrix --> real 8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 66) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 67) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 68) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 69) channel_matrix --> real 4, imag -20
[NR_PHY]   (uI 0, gI 0, pI 70) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 71) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 72) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 73) channel_matrix --> real 0, imag -12
[NR_PHY]   (uI 0, gI 0, pI 74) channel_matrix --> real -4, imag -4
[NR_PHY]   (uI 0, gI 0, pI 75) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 76) channel_matrix --> real -8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 77) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 78) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 79) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 80) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 81) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 82) channel_matrix --> real -4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 83) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 84) channel_matrix --> real 12, imag -20
[NR_PHY]   (uI 0, gI 0, pI 85) channel_matrix --> real -4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 86) channel_matrix --> real 8, imag -8
[NR_PHY]   (uI 0, gI 0, pI 87) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 88) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 89) channel_matrix --> real 8, imag -12
[NR_PHY]   (uI 0, gI 0, pI 90) channel_matrix --> real 0, imag -16
[NR_PHY]   (uI 0, gI 0, pI 91) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 92) channel_matrix --> real 0, imag -4
[NR_PHY]   (uI 0, gI 0, pI 93) channel_matrix --> real 8, imag -4
[NR_PHY]   (uI 0, gI 0, pI 94) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 95) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 96) channel_matrix --> real 4, imag -8
[NR_PHY]   (uI 0, gI 0, pI 97) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 98) channel_matrix --> real 8, imag -16
[NR_PHY]   (uI 0, gI 0, pI 99) channel_matrix --> real 4, imag -12
[NR_PHY]   (uI 0, gI 0, pI 100) channel_matrix --> real 4, imag -16
[NR_PHY]   (uI 0, gI 0, pI 101) channel_matrix --> real 0, imag -8
[NR_PHY]   (uI 0, gI 0, pI 102) channel_matrix --> real 12, imag -8
[NR_PHY]   (uI 0, gI 0, pI 103) channel_matrix --> real 4, imag -12
[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NGAP]   PDUSESSIONSetup initiating message
[NR_RRC]   [gNB 0] gNB_ue_ngap_id 0 
[NR_RRC]   Adding pdusession 0, total nb of sessions 1
00:00:04:00:82:00:0a:0c:40:00:00:00:30:40:00:00:00:00:8b:00:0a:01:f0:ca:cc:3e:9e:00:00:00:3e:00:86:00:01:00:00:88:00:07:00:01:00:00:09:1c:00:
[NR_RRC]   adding rnti e3f8 pdusession 1, nb drb 1, xid 2
[NR_RRC]   recreate existing tunnels, while adding new ones
[GTPU]   try to get a gtp-u not existing output
[NR_RRC]   rrc_gNB_process_NGAP_PDUSESSION_SETUP_REQ : gtpv1u_create_ngu_tunnel failed,start to release UE rnti 58360

Assertion (0) failed!
In cucp_cuup_bearer_context_setup_direct() ../../../openair2/RRC/NR/cucp_cuup_direct.c:195
Unable to configure DRB or to create GTP Tunnel

Exiting execution
../../../openair2/RRC/NR/cucp_cuup_direct.c:195 cucp_cuup_bearer_context_setup_direct() Exiting OAI softmodem: _Assert_Exit_
LS[HW]   [recv] received 16476 samples out of 23040
[HW]   ERROR_CODE_TIMEOUT

[PHY]   rx_rf: Asked for 23040 samples, got 16476 from USRP
[PHY]   problem receiving samples
Exiting ru_thread
```

然后报错的同时手机弹出这个东西：

![image-20230506214315618](C:\Users\cybercolyce\AppData\Roaming\Typora\typora-user-images\image-20230506214315618.png)



还有另一种错误就是没有物理层消息输出，不知道是啥情况：

```
[PHY]   RU 0 RF started opp_enabled 0
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
sleep...
[PHY]   tx_reorder_thread started
[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_PHY]   [gNB 0][RAPROC] Frame 897, slot 19 Initiating RA procedure with preamble 59, energy 55.7 dB (I0 123, thres 120), delay 4 start symbol 0 freq index 0
[MAC]   UL_info[Frame 897, Slot 19] Calling initiate_ra_proc RACH:SFN/SLOT:897/19
[NR_MAC]   Search for not existing rnti (ignore for RA): 384a
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 897 Activating Msg2 generation in frame 898, slot 7 using RA rnti 10b SSB, new rnti 384a index 0 RA index 0
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 898, slotP 7: Generating RA-Msg2 DCI, rnti 0x10b, state 1, CoreSetType 2
[NR_MAC]   [RAPROC] Msg3 slot 17: current slot 7 Msg3 frame 898 k2 7 Msg3_tda_id 3
[NR_MAC]   [gNB 0][RAPROC] Frame 898, Subframe 7: rnti 384a RA state 2
[NR_MAC]   Search for not existing rnti (ignore for RA): 384a
[NR_MAC]   Adding UE with rnti 0x384a
[NR_MAC]   [gNB 0][RAPROC] PUSCH with TC_RNTI 0x384a received correctly, adding UE MAC Context RNTI 0x384a
[NR_MAC]   [RAPROC] RA-Msg3 received (sdu_lenP 7)
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE 384a
[NR_MAC]   Added new CBRA process for UE RNTI 384a with initial CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[RLC]   activated srb0 for UE with RNTI 0x384a
[RLC]   ../../../openair2/LAYER2/nr_rlc/nr_rlc_oai_api.c:693:nr_rlc_add_srb: added srb 1 to UE with RNTI 0x384a
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE 384a
[NR_MAC]   Activating scheduling RA-Msg4 for TC_RNTI 0x384a (state 2)
[NR_MAC]   Unexpected ULSCH HARQ PID 0 (have -1) for RNTI 0x384a (ignore this warning for RA)
[RRC]   initial UL RRC message nr_cellid 0 does not match RRC's 12345678
[NR_RRC]   Decoding CCCH: RNTI 384a, payload_size 6
[NR_RRC]   search by rnti not found 384a
[NR_RRC]   search by rnti not found 384a
[NR_RRC]   Returning new RRC UE context RRC ue id: 0
[NR_RRC]    Created new UE context rnti: 384a, random ue id 5233917362000000, RRC ue id 0
[NR_RRC]   rrc_gNB_generate_RRCSetup for RNTI 384a
[NR_MAC]   DL RRC Message Transfer with 137 bytes for RNTI 384a SRB 0
[NR_MAC]   ( 899. 5) SRB0 has 137 bytes
[NR_MAC]   Generate msg4, rnti: 384a
[NR_MAC]   Encoded RRCSetup Piggyback (137 + 2 bytes), mac_pdu_length 146
[NR_MAC]   (UE RNTI 0x384a) Received Ack of RA-Msg4. CBRA procedure succeeded!
[NR_MAC]   (899.13) Activating RRC processing timer for UE 384a with 10 ms
[NR_MAC]   (900.13) De-activating RRC processing timer for UE 384a
[NR_MAC]   Modified rnti 384a with CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI 384a] [RAPROC] Logical Channel UL-DCCH, processing NR_RRCSetupComplete from UE (SRB1 Active)
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI 384a] UE State = NR_RRC_CONNECTED 
[NGAP]   [gNB 0] Chose AMF 'AMF' (assoc_id 122) through selected PLMN Identity index 0 MCC 466 MNC 92
[NR_PHY]   [gNB 0][RAPROC] Frame 901, slot 19 Initiating RA procedure with preamble 24, energy 43.2 dB (I0 151, thres 120), delay 30 start symbol 0 freq index 0
[NR_PHY]   [gNB 0][RAPROC] Frame 901, slot 19 Initiating RA procedure with preamble 54, energy 40.9 dB (I0 184, thres 120), delay 25 start symbol 4 freq index 0
[NR_PHY]   [gNB 0][RAPROC] Frame 901, slot 19 Initiating RA procedure with preamble 0, energy 42.8 dB (I0 210, thres 120), delay 24 start symbol 8 freq index 0
[MAC]   UL_info[Frame 901, Slot 19] Calling initiate_ra_proc RACH:SFN/SLOT:901/19
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 901 Activating Msg2 generation in frame 902, slot 7 using RA rnti 10b SSB, new rnti b4ef index 0 RA index 0
[NR_MAC]   Search for not existing rnti (ignore for RA): fc2a
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 901 Activating Msg2 generation in frame 902, slot 7 using RA rnti 10f SSB, new rnti fc2a index 0 RA index 1
[MAC]   UL_info[Frame 902, Slot 0] Calling initiate_ra_proc RACH:SFN/SLOT:901/19
[NR_MAC]   Search for not existing rnti (ignore for RA): d99d
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 901 Activating Msg2 generation in frame 902, slot 7 using RA rnti 10b SSB, new rnti d99d index 0 RA index 2
[NR_MAC]   [gNB 0][RAPROC] CC_id 0 Frame 902, slotP 7: Generating RA-Msg2 DCI, rnti 0x10b, state 1, CoreSetType 2
[NR_MAC]   [RAPROC] Msg3 slot 17: current slot 7 Msg3 frame 902 k2 7 Msg3_tda_id 3
[NR_MAC]   [gNB 0][RAPROC] Frame 902, Subframe 7: rnti b4ef RA state 2
[NR_MAC]   nr_generate_Msg2(): cannot find free CCE for RA RNTI 0xfc2a!
[NR_MAC]   nr_generate_Msg2(): cannot find free CCE for RA RNTI 0xd99d!
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   handle harq for rnti b4ef, in RA process
[NR_MAC]   [gNB 0][RAPROC] Frame 903, Slot 10 : CC_id 0 Scheduling retransmission of Msg3 in (903,17)
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   handle harq for rnti b4ef, in RA process
[NR_MAC]   [gNB 0][RAPROC] Frame 904, Slot 10 : CC_id 0 Scheduling retransmission of Msg3 in (904,17)
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   handle harq for rnti b4ef, in RA process
[NR_MAC]   [gNB 0][RAPROC] Frame 905, Slot 10 : CC_id 0 Scheduling retransmission of Msg3 in (905,17)
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   Random Access 0 failed at state 2 (Reached msg3 max harq rounds)
[NR_MAC]   Search for not existing rnti (ignore for RA): b4ef
[NR_MAC]   handle harq for rnti b4ef, in RA process
[NR_MAC]   handle_nr_ul_harq(): unknown RNTI 0xb4ef in PUSCH
[NR_MAC]   Frame.Slot 0.0
UE RNTI 384a (1) PH 49 dB PCMAX 21 dBm, average RSRP -113 (15 meas)
UE 384a: CQI 0, RI 1, PMI (0,0)
UE 384a: dlsch_rounds 14/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.03138 MCS 9
UE 384a: dlsch_total_bytes 1599
UE 384a: ulsch_rounds 372/0/0/0, ulsch_DTX 0, ulsch_errors 0, BLER 0.02824 MCS 9
UE 384a: ulsch_total_bytes_scheduled 31744, ulsch_total_bytes_received 31512
UE 384a: LCID 1: 3 bytes TX

[NR_MAC]   Frame.Slot 128.0
UE RNTI 384a (1) PH 49 dB PCMAX 21 dBm, average RSRP -113 (16 meas)
UE 384a: CQI 0, RI 1, PMI (0,0)
UE 384a: dlsch_rounds 27/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00798 MCS 9
UE 384a: dlsch_total_bytes 3198
UE 384a: ulsch_rounds 756/0/0/0, ulsch_DTX 0, ulsch_errors 0, BLER 0.00718 MCS 9
UE 384a: ulsch_total_bytes_scheduled 64512, ulsch_total_bytes_received 64280
UE 384a: LCID 1: 3 bytes TX

[NR_MAC]   Frame.Slot 256.0
UE RNTI 384a (1) PH 49 dB PCMAX 21 dBm, average RSRP -113 (16 meas)
UE 384a: CQI 0, RI 1, PMI (0,0)
UE 384a: dlsch_rounds 40/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00203 MCS 9
UE 384a: dlsch_total_bytes 4797
UE 384a: ulsch_rounds 1140/0/0/0, ulsch_DTX 0, ulsch_errors 0, BLER 0.00203 MCS 9
UE 384a: ulsch_total_bytes_scheduled 97280, ulsch_total_bytes_received 97048
UE 384a: LCID 1: 3 bytes TX

[NR_MAC]   Frame.Slot 384.0
UE RNTI 384a (1) PH 51 dB PCMAX 21 dBm, average RSRP -113 (16 meas)
UE 384a: CQI 0, RI 1, PMI (0,0)
UE 384a: dlsch_rounds 52/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00057 MCS 9
UE 384a: dlsch_total_bytes 6273
UE 384a: ulsch_rounds 1524/0/0/0, ulsch_DTX 0, ulsch_errors 0, BLER 0.00052 MCS 9
UE 384a: ulsch_total_bytes_scheduled 130048, ulsch_total_bytes_received 129816
UE 384a: LCID 1: 3 bytes TX
```

## 2023/5/14

也许是核心网的问题？成功提取到部分数据的那个核心网用的是师兄部署在云端的k8s的核心网，关键是有这个：

```log
[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NR_MAC]   DL RRC Message Transfer with 28 bytes for RNTI e3f8 SRB 1
[NR_RRC]   Recived RRC GNB UL Information Transfer 
[NR_RRC]   Send RRC GNB UL Information Transfer 
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 110)
[NGAP]   could not found NGAP_ProtocolIE_ID_id_UEAggregateMaximumBitRate
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 71)
[NGAP]   AllowedNSSAI.list.count 1
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 36)
[RRC]   selecting ciphering algorithm 0
[RRC]   selecting integrity algorithm 2
[NR_RRC]   [gNB 0][UE e3f8] Selected security algorithms (0x7fd5c4004c84): 0, 2, changed
[NR_RRC]   [gNB 0][UE e3f8] Saved security key 8BE43001662E75E97CA39F38510C56EC0D2CE0AEC754A8EA968ADD884A249849
[NR_RRC]   
KgNB:8b e4 30 01 66 2e 75 e9 7c a3 9f 38 51 0c 56 ec 0d 2c e0 ae c7 54 a8 ea 96 8a dd 88 4a 24 98 49 
[NR_RRC]   
KRRCenc:6e 8b 94 a6 3c 7c 44 2d a8 9c e5 71 7f 4a 5c 73 
[NR_RRC]   
KRRCint:a7 a0 e3 f0 85 b1 3f ba 85 5d b3 32 27 2c 5e 04 
[NR_RRC]   UE e3f8 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[NR_MAC]   DL RRC Message Transfer with 9 bytes for RNTI e3f8 SRB 1
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] received securityModeComplete on UL-DCCH 1 from UE
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] Logical Channel DL-DCCH, Generate NR UECapabilityEnquiry (bytes 8)
[NR_MAC]   DL RRC Message Transfer with 14 bytes for RNTI e3f8 SRB 1
[NR_RRC]   [FRAME 00000][gNB][MOD 00][RNTI e3f8] received ueCapabilityInformation on UL-DCCH 1 from UE
[NR_RRC]   got UE capabilities for UE e3f8
[NR_RRC]   Send message to ngap: NGAP_UE_CAPABILITIES_IND
[NR_RRC]   SRS configured with 1 ports
[NR_RRC]   do_RRCReconfiguration(): size 224
[NR_RRC]   [gNB 0] Frame 0, Logical Channel DL-DCCH, Generate NR_RRCReconfiguration (bytes 224, UE id e3f8)
[NR_MAC]   DL RRC Message Transfer with 230 bytes for RNTI e3f8 SRB 1
[NR_MAC]   Added new CBRA process for UE RNTI e3f8 with initial CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_MAC]   Activating RRC processing timer for UE e3f8 with 10 ms
[NR_MAC]   (672.19) De-activating RRC processing timer for UE e3f8
[NR_MAC]   Modified rnti e3f8 with CellGroup
[NR_MAC]   Adding SchedulingRequestconfig
[NR_MAC]   Adding BSR config
[NR_MAC]   Adding TAG config
[NR_MAC]   Adding PHR config
[NR_RRC]   Receive RRC Reconfiguration Complete message UE e3f8
```

## 2023/5/18

手机接入了，前面的问题其实是核心网那边没有录入接入手机的key和opc的缘故。现在开始看一下师兄给的参考代码：

```c
Protocol__NRSRSIQEST * nr_srs_est;
nr_srs_est = malloc(sizeof(Protocol__NRSRSIQEST));
protocol__nr__srs__iq__est__init(nr_srs_est);
Protocol__CHANNLEEST ** channel_est;
nr_srs_est->n_channle_est_list  = nr_srs_normalized_channel_iq_matrix.num_ue_srs_ports*nr_srs_normalized_channel_iq_matrix.num_gnb_antenna_elements*nr_srs_normalized_channel_iq_matrix.num_prgs;
            channel_est = malloc(sizeof(Protocol__CHANNLEEST)*nr_srs_est->n_channle_est_list);
```

C语言中`malloc`函数的参数是`size` 是内存块的大小，以字节为单位，该函数返回一个指针 ，指向已分配大小的内存。如果请求失败，则返回 NULL。因此在声明`nr_srs_est`的时候是定义一个指针。`Protocol__CHANNLEEST ** channel_est;`这一句声明的是一个二级指针。为什么是二级指针呢？因为protobuf生成的`.h`文件里就是这么定义的。我们的定义的protobuf文件长这样：

```protobuf
syntax = "proto2";
package protocol;
message NR_SRS_IQ_EST {
  repeated CHANNLE_EST CHANNLE_EST_LIST = 4;
}

message CHANNLE_EST {
  required int32 image = 1;
  required int32 real  = 2;
}
```

然后是protobuf的问题，现在版本的Protobuf好像没有`./autorun`之类的东西，所以用的是protobuf-compiler进行安装的。因此在编成哪个protobuf-c文件后可能会在你include生成的c头文件的时候出现找不到protobuf/protbuf-c.c的问题，我的解决方法是直接下载protobuf-c到OAI中，然后在CMakefile中添加：

```cmake
include_directories(${OPENAIR_DIR}/protobuf-c)
```

但是还会出现这样的问题：

![image-20230522143014404](https://s2.loli.net/2023/05/22/xXKlsedUiv42mwL.png)

我也不懂了

### 2023/5/22

今天重要进展！他妈的把这个CMakefile写完了，真尼玛的弱智

这个由于用了protobuf-compiler，导致这个protobuf-c的include很神秘，我的解决方法是直接git clone下整个prorobuf-c的源码到openairinterface的根目录下，然后添加对应的编译文件：

```cmake
add_executable(nr-softmodem
  ${rrc_h}
  ${nr_rrc_h}
  ${s1ap_h}
#  ${OPENAIR_BIN_DIR}/messages_xml.h
  ${OPENAIR_DIR}/executables/nr-gnb.c
  ${OPENAIR_DIR}/executables/nr-ru.c
  ${OPENAIR_DIR}/executables/nr-softmodem.c
  ${OPENAIR_DIR}/executables/softmodem-common.c
  ${OPENAIR_DIR}/radio/COMMON/common_lib.c
  ${OPENAIR_DIR}/radio/COMMON/record_player.c
  ${OPENAIR2_DIR}/RRC/NAS/nas_config.c
  ${OPENAIR2_DIR}/RRC/NAS/rb_config.c
  ${OPENAIR_DIR}/common/utils/lte/ue_power.c
  ${OPENAIR_DIR}/common/utils/lte/prach_utils.c
  ${OPENAIR_DIR}/common/utils/nr/nr_common.c
  ${PHY_INTERFACE_DIR}/queue_t.c
  ${OPENAIR1_DIR}/PHY/TOOLS/phy_scope_interface.c
  ${T_SOURCE}
  ${SHLIB_LOADER_SOURCES}
  ${OPENAIR1_DIR}/SCHED_NR/MESSAGES/srs-info.pb-c.h
  ${OPENAIR1_DIR}/SCHED_NR/MESSAGES/srs-info.pb-c.c
  ${OPENAIR_DIR}/protobuf-c/protobuf-c/protobuf-c.c
  ${OPENAIR_DIR}/protobuf-c/protobuf-c/protobuf-c.h
  )
```

最下面那4行加上去然后编译。

