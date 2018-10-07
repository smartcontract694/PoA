contract Vid_Traceability{

address public owner;
Video public thisVideo;
bytes32 public agreementFormIPFS;//it holds the details of the agreement between the editors and the owner

//this structure is used for all videos in a chain used for provenance data
struct Video {
  address vid_owner;//Ethereum address of the video owner
  string info;//informtation about the video
  bytes32 IPFS_Hash;//IPFS hash of the uploaded video on the IPFS server
  address SC_address;//address of Smart contract
  string metadata;
  uint256 timestamp;
}

enum artistState {SentRequest,  GrantedPermission, DeniedPermission, SentAttestationRequest, GrantedAttestation, DeniedAttestation } 


//an Artist can have multiple requests each with a unique IPFS hash
struct Artist{
    artistState state;
    address EA;
    bytes32 hash;
    bool result;//by default false unless granted permission
}

bool parent;//if true it means it has a parent and it is a child
Video public parent_video;//parent video if any

mapping (bytes32 =>Video) public Granted_Permission_ChildVideos;//for history tracking, mapping all children videos with their SC address
mapping (bytes32 => Artist) public Granted_Permissions;// EA of artists with granted permissions, maps between the EA and the IPFS hash of the video
mapping (bytes32 => Artist) public Denied_Permissions;//list of artists with denied permissions,maps between the EA and the IPFS hash of the video
mapping (bytes32=>Artist) public Requests;//all requets , EA and IPFS hash


modifier OnlyOwner(){
    require(msg.sender == owner);
    _;
} 

modifier NotOwner(){
    require(msg.sender != owner);
    _;
}
//events
event ArtistRequestingPermission(address artist);
event ArtistRequestRegistered(address artist, bytes32 IPFS_Hash);
event PermissionGranted(string info, address artist);
event PermissionDenied(string info, address artist);
event AttestationRequest(string info, address Scaddress);
event AttestationGranted(string info, address Scaddress);
event AttestationDenied(string info, address Scaddress);

//constructor
function Vid_Traceability(){
    owner = msg.sender;
    parent = false;//by default false
    thisVideo.vid_owner = msg.sender;
    thisVideo.IPFS_Hash = 'kkowfwjf';
    thisVideo.info = "A comic Video";
    thisVideo.SC_address = this;
    thisVideo.metadata = "Data:18/9/2018, Time:5:57";
    thisVideo.timestamp = block.timestamp;
    agreementFormIPFS = 'jhfghffs';
}


  //this function updates the parent information for traceability 
    function updateProvenanceData(string information, bytes32 hash, address SC, address EA) OnlyOwner{
        parent = true;//yes it has a parent
        parent_video.vid_owner = EA;
       parent_video.info = information;
        parent_video.IPFS_Hash = hash;
        parent_video.SC_address = SC;
    }
    
    //this function is called by other interested editors to create new versions of the video
    //an editor requetsing permissions indicates that they agreed to the agreement form
    function requestPersmission(bytes32 IPFShash) NotOwner{
        require(Requests[IPFShash].hash != IPFShash);//to ensure this video is a new request
        ArtistRequestingPermission(msg.sender);//register artist
        Requests[IPFShash].state = artistState.SentRequest;
        Requests[IPFShash].EA = msg.sender;
        Requests[IPFShash].hash = IPFShash;
        Requests[IPFShash].result = false;//false by default until granted permission
        ArtistRequestRegistered(msg.sender, IPFShash);
    }
    
    //this function is used by the owner to grant permsissions
    function grantPermission(bool result, address artist,bytes32 IPFShash) OnlyOwner{
        require(Requests[IPFShash].state == artistState.SentRequest);
        if(result){
            Granted_Permissions[IPFShash].EA = artist;
            Granted_Permissions[IPFShash].state = artistState.GrantedPermission;
            Granted_Permissions[IPFShash].result = result;
            Granted_Permissions[IPFShash].hash = IPFShash;
            Requests[IPFShash].state == artistState.GrantedPermission;
            Requests[IPFShash].result = true;
            PermissionGranted("Permission Granted to address ", artist);
            
        }
        else {
             Denied_Permissions[IPFShash].EA = artist;
             Denied_Permissions[IPFShash].state = artistState.DeniedPermission;
             Denied_Permissions[IPFShash].result = result;
             Denied_Permissions[IPFShash].hash = IPFShash;
             Requests[IPFShash].state == artistState.DeniedPermission;
            PermissionDenied("Permission Denied to address ", artist);
        }
    }
    
    //this function is called by the artist after getting an approval and creating a child SC 
    function AttestSC(string infor, bytes32 hash, string meta, address SCaddress) NotOwner{
        require(Granted_Permissions[hash].state == artistState.GrantedPermission);//requires state to be granted
        AttestationRequest("Address of child: " , SCaddress);
        Granted_Permissions[hash].state = artistState.SentAttestationRequest;
    }
    
    function GrantAttestation(bool result, string infor, bytes32 hash, string meta, address SCaddress)OnlyOwner{
    require(Granted_Permissions[hash].state == artistState.SentAttestationRequest);
        if(result){
         Granted_Permission_ChildVideos[hash].vid_owner = msg.sender;//save all info as new entry
         Granted_Permission_ChildVideos[hash].info = infor;
         Granted_Permission_ChildVideos[hash].IPFS_Hash = hash;
         Granted_Permission_ChildVideos[hash].SC_address = SCaddress;
         Granted_Permission_ChildVideos[hash].timestamp = block.timestamp;
         Granted_Permission_ChildVideos[hash].metadata = meta;
         Granted_Permissions[hash].state = artistState.GrantedAttestation;
         Requests[hash].state = artistState.GrantedAttestation;
         AttestationGranted("Successfully Attested: ", SCaddress);
        }
        else
        {
         Granted_Permissions[hash].state = artistState.DeniedAttestation;//Granted permission for the request, but denied attestation
         Requests[hash].state = artistState.DeniedAttestation;
         AttestationDenied("Denied Attestation: ", SCaddress);   
        }
    }
    
}
