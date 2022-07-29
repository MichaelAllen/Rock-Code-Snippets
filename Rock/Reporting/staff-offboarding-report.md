---
tags:
    - language/sql
    - language/lava
    - type/reporting
date created: 2022-07-29 10:25:44
date modified: 2022-07-29 13:45:05
---

# Staff Off-Boarding Report

This is a fairly complicated report that attempts to show everything that you would need to change when someone transitions off of your staff team. It also, when possible, provides links to the edit pages of the various things that need to be changed.

It looks in the following places:

1. Security Group Membership
2. Room Reservation Approval Groups
3. Connector Group Memberships
4. Group Leadership (All Group Types)
5. Default Connector
6. Active Connections
7. Active Event Contact
8. Active Registration Contact
9. Org Chart Group Membership
10. Miscellaneous Security Authorizations
11. SMS From Values
12. Room Reservations (Admin contact or Event contact)
13. Service Job Notifications
14. Communication Templates (From/CC/BCC)
15. System Communications (From/CC/BCC)
16. Assigned Workflows
17. Assigned Projects
18. Assigned Project Tasks
19. Watching Projects

There are several "magic numbers" in these queries. You will want to check all of the Entity IDs and Page numbers against your own system and update as needed.

## Page Parameter Filter Block

| Field Type | Key    |
| ---------- | ------ |
| Person     | Person |

## Dynamic Data Block

### Parameters

`Person='00000000-0000-0000-0000-000000000000'`

### Query

```sql
-- Get their Person Id
DECLARE @PersonId int = ( SELECT [PersonId] FROM [PersonAlias] WHERE [Guid] = CAST( @Person AS uniqueidentifier ) );

-- Get all of their PersonAlias IDs
SELECT pa.[Id] INTO #PersonAliasIds FROM [PersonAlias] pa WHERE pa.[PersonId] = @PersonId;

-- Get their email
DECLARE @PersonEmail varchar(max) = ( SELECT NULLIF( [Email], '' ) FROM [Person] WHERE [Id] = @PersonId );

-- 1. Security Group Membership
SELECT
    g.[Id]
    ,g.[Name] 'Group'
    ,g.[IsActive]
    ,g.[IsArchived]
    ,gs.[SyncDataViewId]
FROM
    [Group] g
    INNER JOIN [GroupMember] gm ON
        g.[Id] = gm.[GroupId]
        AND gm.[IsArchived] != 1
    LEFT JOIN [GroupSync] gs ON g.[Id] = gs.[GroupId]
WHERE
    gm.[PersonId] = @PersonId
    AND g.[IsSecurityRole] = 1
ORDER BY g.[Name]
;

-- 2. Room Reservation Approval Groups
SELECT
    g.[Id]
    ,g.[Name] 'Group'
    ,g.[IsActive]
    ,g.[IsArchived]
    ,gs.[SyncDataViewId]
FROM
    [Group] g
    INNER JOIN [GroupMember] gm ON
        g.[Id] = gm.[GroupId]
        AND gm.[IsArchived] != 1
    LEFT JOIN [GroupSync] gs ON g.[Id] = gs.[GroupId]
WHERE
    gm.[PersonId] = @PersonId
    AND g.[GroupTypeId] = 82
ORDER BY g.[Name]
;

-- 3. Connector Group Memberships
SELECT
    g.[Id]
    ,g.[Name] 'Group'
    ,g.[IsActive]
    ,g.[IsArchived]
    ,gs.[SyncDataViewId]
FROM
    [Group] g
    INNER JOIN [GroupMember] gm ON
        g.[Id] = gm.[GroupId]
        AND gm.[IsArchived] != 1
    LEFT JOIN [GroupSync] gs ON g.[Id] = gs.[GroupId]
WHERE
    gm.[PersonId] = @PersonId
    AND g.[ParentGroupId] = 60
ORDER BY g.[Name]
;

-- 4. Group Leadership (All Group Types)
SELECT
    g.[Id]
    ,g.[Name] 'Group'
    ,g.[IsActive]
    ,g.[IsArchived]
FROM
    [Group] g
    INNER JOIN [GroupMember] gm ON
        g.[Id] = gm.[GroupId]
        AND gm.[IsArchived] != 1
    INNER JOIN [GroupTypeRole] gtr ON
        gm.[GroupRoleId] = gtr.[Id]
WHERE
    gm.[PersonId] = @PersonId
    AND gtr.[IsLeader] = 1
ORDER BY g.[Name]
;

-- 5. Default Connector
SELECT
    co.[Id]
    ,co.[ConnectionTypeId]
    ,co.[Name]
    ,c.[Name] 'CampusName'
FROM
    [ConnectionOpportunity] co 
    INNER JOIN [ConnectionOpportunityCampus] coc ON co.[Id] = coc.[ConnectionOpportunityId]
    INNER JOIN [Campus] c ON coc.[CampusId] = c.[Id]
    INNER JOIN #PersonAliasIds pa ON coc.[DefaultConnectorPersonAliasId] = pa.[Id]
ORDER BY
    co.Name
    ,c.Name
;

-- 6. Active Connections
SELECT
    cr.[Id]
    ,co.[Name] 'ConnectionOpportunity'
    ,p.[NickName]+' '+p.[LastName] 'PersonName'
FROM
    [ConnectionRequest] cr 
    INNER JOIN [ConnectionOpportunity] co ON cr.[ConnectionOpportunityId] = co.[Id]
    INNER JOIN [PersonAlias] pa ON cr.[PersonAliasId] = pa.[Id]
    INNER JOIN [Person] p ON pa.[PersonId] = p.[Id]
    INNER JOIN #PersonAliasIds pa2 ON cr.[ConnectorPersonAliasId] = pa2.[Id]
WHERE
    cr.[ConnectionState] = 0
ORDER BY
    co.[Name]
    ,cr.[Id]
;

-- 7. Active Event Contact
SELECT
    eio.[Id]
    ,ei.[Name]
FROM
    [EventItem] ei
    INNER JOIN [EventItemOccurrence] eio ON ei.[Id] = eio.[EventItemId]
    INNER JOIN [Schedule] s ON eio.[ScheduleId] = s.[Id]
    INNER JOIN #PersonAliasIds pa ON eio.[ContactPersonAliasId] = pa.[Id]
WHERE
    ei.[IsActive] = 1
    AND s.[EffectiveEndDate] > GETDATE()
;

-- 8. Active Registration Contact
SELECT
    ri.[Id]
    ,ri.[Name]
FROM
    [RegistrationInstance] ri
    INNER JOIN [Registrationtemplate] rt on ri.[RegistrationTemplateId] = rt.[Id]
    INNER JOIN #PersonAliasIds pa ON ri.[ContactPersonAliasId] = pa.[Id]
WHERE
    ri.[IsActive] = 1
    AND ri.[EndDateTime] > GETDATE()
;

-- 9. Org Chart Group Membership
SELECT
    g.[Id]
    ,g.[Name] 'Group'
    ,g.[IsActive]
    ,g.[IsArchived]
    ,gs.[SyncDataViewId]
FROM
    [Group] g
    INNER JOIN [GroupMember] gm ON
        g.[Id] = gm.[GroupId]
        AND gm.[IsArchived] != 1
    LEFT JOIN [GroupSync] gs ON g.[Id] = gs.[GroupId]
WHERE
    gm.[PersonId] = @PersonId
    AND g.[GroupTypeId] = 28
ORDER BY g.[Name]
;

-- 10. Miscellaneous Security Settings
SELECT DISTINCT
    p.[InternalName] 'PageName'
    ,g.[Name] 'GroupName'
    ,a.[EntityTypeId] 'EntityType'
    ,et.[FriendlyName] 'EntityName'
    ,a.[EntityId] 'EntityId'
    ,rt.[Name] 'RegName'
    ,dv.[Name] 'Dataview'
    ,r.[Name] 'Report'
    ,b.[Name] 'Block'
    ,b.[PageId]
    ,s.[Name] 'Schedule'
FROM
    [Auth] a 
    INNER JOIN [PersonAlias] pa ON
        a.PersonAliasId = pa.Id
        AND pa.Guid = @person
    INNER JOIN [EntityType] et ON
        a.EntityTypeId = et.Id
    LEFT JOIN [Page] p ON
        a.[EntityId] = p.[Id]
        AND et.[Id] = 2
    LEFT JOIN [Block] b ON
        a.[EntityId] = b.[Id]
        AND et.[Id] = 9
    LEFT JOIN [Group] g ON
        a.[EntityId] = g.[Id]
        AND et.[Id] = 16
    LEFT JOIN [DataView] dv ON
        a.[EntityId] = dv.[Id]
        AND et.[Id] = 34
    LEFT JOIN [Schedule] s ON
        a.[EntityId] = s.[Id]
        AND et.[Id] = 54
    LEFT JOIN [Report] r ON
        a.[EntityId] = r.[Id]
        AND et.[Id] = 107
    LEFT JOIN [RegistrationTemplate] rt ON
        a.[EntityId] = rt.[Id]
        AND et.[Id] = 234
ORDER BY et.[FriendlyName]
;

-- 11. SMS From Values
SELECT
    dv.[Value]
    ,dv.[Description]
FROM
    [DefinedValue] dv
    INNER JOIN [AttributeValue] av ON
        dv.[Id] = av.[EntityId]
        AND av.[AttributeId] = 949
    INNER JOIN [PersonAlias] pa ON av.[Value] = pa.[Guid]
    INNER JOIN #PersonAliasIds pa2 ON pa.[Id] = pa2.[Id]
WHERE [DefinedTypeId] = 32
;


-- 12. Room Reservations (Admin contact or Event contact)
SELECT
    r.[Id]
    ,r.[Name]
FROM
    [_com_bemaservices_RoomManagement_Reservation] r
WHERE
    r.[LastOccurrenceEndDateTime] > GETDATE()
    AND (
        r.[EventContactPersonAliasId] IN ( SELECT [Id] FROM #PersonAliasIds )
        OR r.[AdministrativeContactPersonAliasId] IN ( SELECT [Id] FROM #PersonAliasIds )
    )
;

-- 13. Service Job Notifications
SELECT 
    j.[IsActive]
    ,j.[Id]
    ,j.[Name]
    ,j.[Class]
FROM [ServiceJob] j
WHERE
    @PersonEmail IS NOT NULL
    AND j.[NotificationEmails] LIKE '%' + @PersonEmail + '%'
;

-- 14. Communication Templates (From/CC/BCC)
SELECT 
    ct.[Id]
    ,ct.[Name]
    ,ct.[IsActive]
FROM [CommunicationTemplate] ct
WHERE
    @PersonEmail IS NOT NULL
    AND (
        ct.[FromEmail] LIKE '%' + @PersonEmail + '%'
        OR ct.[CCEmails] LIKE '%' + @PersonEmail + '%'
        OR ct.[BCCEmails] LIKE '%' + @PersonEmail + '%'
    )
;

-- 15. System Communications (From/CC/BCC)
SELECT 
    sc.[Id]
    ,sc.[Title]
    ,sc.[IsActive]
FROM [SystemCommunication] sc
WHERE
    @PersonEmail IS NOT NULL
    AND (
        sc.[From] LIKE '%' + @PersonEmail + '%'
        OR sc.[To] LIKE '%' + @PersonEmail + '%'
        OR sc.[CC] LIKE '%' + @PersonEmail + '%'
        OR sc.[BCC] LIKE '%' + @PersonEmail + '%'
    )
;

-- 16. Assigned Workflows
SELECT
    w.[Id]
    ,w.[Name]
    ,wt.[Name] 'WorkflowType'
    ,wat.[Name] 'ActivityType'
FROM
    [Workflow] w
    INNER JOIN [WorkflowType] wt ON w.[WorkflowTypeId] = wt.[Id]
    INNER JOIN [WorkflowActivity] wa ON wa.[WorkflowId] = w.[Id]
    INNER JOIN [WorkflowActivityType] wat ON wa.[ActivityTypeId] = wat.[Id]
    INNER JOIN #PersonAliasIds pa ON pa.[Id] = wa.[AssignedPersonAliasId]
WHERE w.[CompletedDateTime] IS NULL
;

-- 17. Assigned Projects
SELECT
    j.[Id]
    ,j.[Name]
FROM
    [_com_blueboxmoon_ProjectManagement_ProjectAssignee] ja
    INNER JOIN [_com_blueboxmoon_ProjectManagement_Project] j ON ja.[ProjectId] = j.[Id]
    INNER JOIN #PersonAliasIds pa ON ja.[PersonAliasId] = pa.[Id]
WHERE j.[IsActive] = 1
;

-- 18. Assigned Project Tasks
SELECT
    t.[Name]
    ,j.[Id] 'ProjectId'
    ,j.[Name] 'ProjectName'
FROM
    [_com_blueboxmoon_ProjectManagement_Task] t
    INNER JOIN [_com_blueboxmoon_ProjectManagement_Project] j ON t.[ProjectId] = j.[Id]
    INNER JOIN #PersonAliasIds pa ON t.[AssignedToPersonAliasId] = pa.[Id]
WHERE
    t.[IsActive] = 1
    AND j.[IsActive] = 1
;

-- 19. Watching Projects
SELECT
    j.[Id]
    ,j.[Name]
FROM
    [_com_blueboxmoon_ProjectManagement_Watching] jw
    JOIN [_com_blueboxmoon_ProjectManagement_Project] j ON jw.[ProjectId] = j.[Id]
    INNER JOIN #PersonAliasIds pa ON jw.[PersonAliasId] = pa.[Id]
WHERE
    j.[IsActive] = 1
    AND jw.[IsWatching] = 1
;
```

### Display Lava

```liquid
{% assign Set = 'Global' | PageParameter:'Person' %}
{% if Set != '' %}
    {% assign Person2 = 'Global' | PageParameter:'Person' | PersonByAliasGuid %}

    <h2>{{Person2.FullName}}</h2>
    {% if Person2.Email contains '@valorouschurch.com' %}
        <h3>
            <a href="/Person/{{ Person2.Id}}" class="btn btn-default"><i class="fa fa-pencil"></i></a>
            Email: {{Person2.Email }}
            <br><small>NOTE: Update their email last. Some of these searches are based on email.</small>
        </h3>
    {% endif %}
    <br>
    <div class="row">

    {% comment %} SQL Table 1 - Security Group Membership {% endcomment %}
        {% assign results = table1.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Security Group Membership' ]}
                {% for row in table1.rows %}
                    {% if row.SyncDataViewId <> '' and row.SyncDataViewId %}
                        {% capture editurl %}/page/145?DataViewId={{ row.SyncDataViewId }}{% endcapture %}
                    {% else %}
                        {% capture editurl %}/page/113?GroupId={{ row.Id }}{% endcapture %}
                    {% endif %}
                    <b {% if row.IsActive != true or row.IsArchived != false %}style="color:Lightgrey;"{% endif %}><a href="{{ editurl }}" class="btn-xs btn-default">{% if row.SyncDataViewId <> '' and row.SyncDataViewId %}<i class="fa fa-sync-alt"></i>{%else%}<i class="fa fa-pencil"></i>{% endif %}</a> {{ row.Group }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 2 - Room Reservation Approval Groups {% endcomment %}
        {% assign results = table2.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Room Reservation Approval Groups']}
                {% for row in table2.rows %}
                    {% if row.SyncDataViewId <> '' and row.SyncDataViewId %}
                        {% capture editurl %}/page/145?DataViewId={{ row.SyncDataViewId }}{% endcapture %}
                    {% else %}
                        {% capture editurl %}/page/113?GroupId={{ row.Id }}{% endcapture %}
                    {% endif %}
                    <b {% if row.IsActive != true or row.IsArchived != false %}style="color:Lightgrey;"{% endif %}><a href="/page/113?GroupId={{ row.Id }}" class="btn-xs btn-default">{% if row.SyncDataViewId <> '' and row.SyncDataViewId %}<i class="fa fa-sync-alt"></i>{% else %}<i class="fa fa-pencil"></i>{% endif %}</a> {{ row.Group }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 3 - Connector Group Memberships {% endcomment %}
        {% assign results = table3.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Connector Group Memberships']}
                {% for row in table3.rows %}
                    {% if row.SyncDataViewId <> '' and row.SyncDataViewId %}
                        {% capture editurl %}/page/145?DataViewId={{ row.SyncDataViewId }}{% endcapture %}
                    {% else %}
                        {% capture editurl %}/page/113?GroupId={{ row.Id }}{% endcapture %}
                    {% endif %}
                    <b {% if row.IsActive != true or row.IsArchived != false %}style="color:Lightgrey;"{% endif %}><a href="/page/113?GroupId={{ row.Id }}" class="btn-xs btn-default">{% if row.SyncDataViewId <> '' and row.SyncDataViewId %}<i class="fa fa-sync-alt"></i>{% else %}<i class="fa fa-pencil"></i>{% endif %}</a> {{ row.Group }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 4 - Group Leadership {% endcomment %}
        {% assign results = table4.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Group Leadership' ]}
                {% for row in table4.rows %}
                    <b {% if row.IsActive != true or row.IsArchived != false %}style="color:Lightgrey;"{% endif %}><a href="/page/113?GroupId={{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Group }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 5 - Default Connector {% endcomment %}
        {% assign results = table5.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Default Connector' ]}
                {% for row in table5.rows %}
                    <b><a href="/page/411?ConnectionOpportunityId={{ row.Id }}&ConnectionTypeId={{ row.ConnectionTypeId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }} | {{ row.CampusName }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 6 - Active Connections {% endcomment %}
        {% assign results = table6.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Active Connections' ]}
                {% for row in table6.rows %}
                    <b><a href="/page/408?ConnectionRequestId={{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.ConnectionOpportunity }} | {{ row.PersonName }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 7 - Active Event Contact {% endcomment %}
        {% assign results = table7.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Active Event Contact' ]}
                {% for row in table7.rows %}
                    <b><a href="/page/402?EventItemOccurrenceId={{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 8 - Active Registration Contact {% endcomment %}
        {% assign results = table8.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Active Registration Contact' ]}
                {% for row in table8.rows %}
                    <b><a href="/RegistrationInstance/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 9 - Org Chart Group Membership {% endcomment %}
        {% assign results = table9.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Org Chart Group Membership' ]}
                {% for row in table9.rows %}
                    {% if row.SyncDataViewId <> '' and row.SyncDataViewId %}
                        {% capture editurl %}/page/145?DataViewId={{ row.SyncDataViewId }}{% endcapture %}
                    {% else %}
                        {% capture editurl %}/page/113?GroupId={{ row.Id }}{% endcapture %}
                    {% endif %}
                    <b {% if row.IsActive != true or row.IsArchived != false %}style="color:Lightgrey;"{% endif %}><a href="{{ editurl }}" class="btn-xs btn-default">{% if row.SyncDataViewId <> '' and row.SyncDataViewId %}<i class="fa fa-sync-alt"></i>{% else %}<i class="fa fa-pencil"></i>{% endif %}</a> {{ row.Group }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 10 - Miscellaneous Security Authorizations {% endcomment %}
        {% assign results = table10.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Miscellaneous Authorizations' ]}
                {% for row in table10.rows %}
                    {% case row.EntityType %}
                    {% when '2' %}
                        <b><a href="/page/103?Page={{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.PageName }} ({{row.EntityName}})</b><br>
                    {% when '9' %}
                        <b><a href="/page/103?Page={{ row.PageId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Block }} ({{row.EntityName}})</b><br>
                    {% when '16' %}
                        <b><a href="/page/113?GroupId={{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.GroupName }} ({{row.EntityName}})</b><br>
                    {% when '34' %}
                        <b><a href="/page/145?DataViewId={{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Dataview }} ({{row.EntityName}})</b><br>
                    {% when '54' %}
                        <b><a href="/Schedules?ScheduleId={{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Schedule }} ({{row.EntityName}})</b><br>
                    {% when '107' %}
                        <b><a href="/page/149?ReportId={{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Report }} ({{row.EntityName}})</b><br>
                    {% when '234' %}
                        <b><a href="/RegistrationInstance/{{ row.EntityId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.RegName }} ({{row.EntityName}})</b><br>
                    {% else %}
                        <b>{{ row.EntityName }} ({{ row.EntityType }}) | Id:{{ row.EntityId }}</b><br>
                    {% endcase %}
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 11 - SMS From Values {% endcomment %}
        {% assign results = table11.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'SMS From Values' ]}
                {% for row in table11.rows %}
                    <b><a href="/page/327" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Value }} | {{ row.Description }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 12 - Room Reservations {% endcomment %}
        {% assign results = table12.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Room Reservations']}
                {% for row in table12.rows %}
                    <b><a href="/ReservationDetail?ReservationId={{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 13 - Service Job Notifications {% endcomment %}
        {% assign results = table13.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Service Job Notifications' ]}
                {% for row in table13.rows %}
                    <b {%if row.IsActive !=1 %}style="color:Lightgrey;"{% endif %}><a href="/page/115?serviceJobId={{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ rowName }} ({{ row.Class }})</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 14 - Communication Templates {% endcomment %}
        {% assign results = table14.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Communication Templates' ]}
                {% for row in table14.rows %}
                    <b {%if row.IsActive !=1 %}style="color:Lightgrey;"{% endif %}><a href="/admin/communications/templates/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 15 - System Communications {% endcomment %}
        {% assign results = table15.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'System Communications' ]}
                {% for row in table15.rows %}
                    <b {%if row.IsActive !=1 %}style="color:Lightgrey;"{% endif %}><a href="/communications/system/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Title }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 16 - Assigned Workflows {% endcomment %}
        {% assign results = table16.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Assigned Workflows' ]}
                {% for row in table16.rows %}
                    <b><a href="/workflow/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }} ({{ row.WorkflowType }} - {{ row.ActivityType }})</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 17 - Assigned Projects {% endcomment %}
        {% assign results = table17.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Assigned Projects' ]}
                {% for row in table17.rows %}
                    <b><a href="/project/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 18 - Assigned Project Tasks {% endcomment %}
        {% assign results = table18.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Assigned Project Tasks' ]}
                {% for row in table18.rows %}
                    <b><a href="/project/{{ row.ProjectId }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }} ({{ row.ProjectName }})</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    {% comment %} SQL Table 19 - Watching Projects {% endcomment %}
        {% assign results = table19.rows | Size %}
        {% if results != 0 %}
            <div class="col-md-4">
                {[ panel title:'Watching Projects' ]}
                {% for row in table19.rows %}
                    <b><a href="/project/{{ row.Id }}" class="btn-xs btn-default"><i class="fa fa-pencil"></i></a> {{ row.Name }}</b><br>
                {% endfor %}
                {[ endpanel ]}
            </div>
        {% endif %}

    </div> 
{% else %}
    <h2>Select a Person Above</h2>
{% endif %}
```
