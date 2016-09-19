# approval-comments-ui-action-snow

UI Action - Approve
Action name - commentsDialog
Active - True
Client - True
List context menu - True
Onclick - commentsDialog()
Condition - isApprovalMine(current)

Script - 

function commentsDialog() {	
	//Get the values to pass into the dialog
	var sysId = g_sysId; 
	if (typeof rowSysId == 'undefined')  
	   sysId = gel('sys_uniqueValue').value;  
	else  
	   sysId = rowSysId;
	
	var gr = new GlideRecord('sysapproval_approver');
	gr.addQuery('sys_id', sysId);
	gr.query();
	while(gr.next()){
		var reqGr = new GlideRecord('sc_req_item');
		reqGr.addQuery('sys_id',gr.sysapproval);
		//reqGr.addQuery('cat_item','d2dc2d810fdcaa002c1bcdbce1050e02');
		reqGr.query();
		if(reqGr.hasNext()){
			//Initialize the GlideDialogWindow
			var gdw = new GlideDialogWindow('add_approval_dialog');
			gdw.setTitle('Add Approval Comments');
			gdw.setPreference('sysparm_sysID', sysId);
			gdw.setPreference('sysparm_table', 'sysapproval_approver');
 
			//Open the dialog window
			gdw.render();
		}
		else {
			gr.state = "approved";
			gr.update();
			window.location = window.location.href;
		}
	}
}


UI Page - add_approval_dialog

HTML - 

<?xml version="1.0" encoding="utf-8" ?>
<j:jelly trim="false" xmlns:j="jelly:core" xmlns:g="glide" xmlns:j2="null" xmlns:g2="null">
<g:ui_form>
<input type="hidden" id="cancelled" name="cancelled" value="false"/>
<input type="hidden" id="approvalSysID" name="approvalSysID" value="${sysparm_sysID}"/>
<table width="100%" cellpadding="0" cellspacing="0">
 <tr><td>
	 <g:ui_multiline_input_field label="Comments" name="comments" id="comments" mandatory="true"/></td></tr>
</table>  
<table width="100%" cellpadding="0" cellspacing="0">
 <tr><td align="right" nowrap="true"><br /><g:dialog_buttons_ok_cancel ok="return validateForm();" cancel="return onCancel();"/></td></tr>
</table>
</g:ui_form>
</j:jelly>

Client script -

function onCancel() {
 var c = gel('cancelled');
 c.value = "true";
 GlideDialogWindow.get().destroy();
 return false;
}
function validateForm() {
	var approvalComments = gel('comments').value;
	approvalComments = trim(approvalComments);
 if (approvalComments == '') {
alert("${JS:gs.getMessage('Please input approval comments')}");
return false;
 }
else {
 return true;
}
}

Processing script -

if (cancelled == "false") {
var grNewUser = new GlideRecord ("sysapproval_approver");
grNewUser.addQuery("sys_id",approvalSysID);
grNewUser.query();
	while(grNewUser.next()) {
grNewUser.state="approved";
grNewUser.comments=request.getParameter("comments");
grNewUser.update();
	}
gs.addInfoMessage("Request is "+grNewUser.state+".");
response.sendRedirect("sysapproval_approver.do?sys_id=" + approvalSysID);
}
