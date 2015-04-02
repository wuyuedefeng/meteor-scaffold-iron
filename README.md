# meteor-scaffold-iron
[英文参考文档](https://medium.com/@s_eschweiler/how-to-build-web-apps-ultra-fast-with-meteor-iron-scaffolding-and-automatic-form-generation-11734eda8e67)

## 使用到的package
* twbs:bootstrap
* aldeed:collection2
* aldeed:autoform
* aldeed:delete-button
* momentjs:moment
* ian:accounts-ui-bootstrap-3
* natestrauser:font-awesome
* momentjs:moment

`iron add [package-name]`

```
$ iron remove autopublish
$ iron remove insecure
```

## 生成collection
`iron g:collection issues`

## 生成controller  template  router
```
$ iron g:controller Issue
$ iron g:template InsertIssue
$ iron g:template IssueList
$ iron g:template AccessDenied
$ iron g:route '/insert_issue'
$ iron g:route '/issues_list'
$ iron g:route '/issues/:_id'
```

# autoform
## 定义
```issues.js
Issues = new Mongo.Collection('issues');

Issues.attachSchema(new SimpleSchema({
  title:{
    type: String,
    label: "Title",
    max: 200
  },
  description:{
    type: String,
    label: "Description",
    max: 1024
  },
  dueDate:{
    type: Date,
    label: "Due Date",
    optional: true
  },
  priority:{
    type: String,
    label: "Priority",
    allowedValues: ['High', 'Medium', 'Low'],
    optional: true
  },
  createdBy: {
    type: String,
    autoValue: function() {
       return this.userId
    }
  }
}));

if (Meteor.isServer) {
  Issues.allow({
    insert: function (userId, doc) {
      return true;
    },

    update: function (userId, doc, fieldNames, modifier) {
      return true;
    },

    remove: function (userId, doc) {
      return true;
    }
  });
}
```
```insert_issue.html
<template name="InsertIssue">
    <h1>Create New Issue</h1>
    {{> quickForm collection="Issues" id="insertIssueForm" type="insert" omitFields="createdBy" buttonContent="Create"}}
</template>
```edit_issue.html
<template name="EditIssue">
    <h1>Edit Issue</h1>
    {{> quickForm collection="Issues" doc=this id="editIssueForm" type="update" omitFields="createdBy" buttonContent="Update"}}
    <hr>
    {{> quickRemoveButton collection="Issues" _id=this._id beforeRemove=beforeRemove class="btn btn-danger"}}
</template>
```edit_issue.js
Template.EditIssue.helpers({
    beforeRemove: function () {
        return function (collection, id) {
            var doc = collection.findOne(id);
            if (confirm('Really delete issue:"' + doc.title + '"?')) {
                this.remove();
                Router.go('issuesList');
            }
        };
    }
});
```
```issues_list.html 用于显示
<template name="IssuesList">
    <h1>Issues List</h1>
    <table class="table table-hover">
        <thead>
        <tr>
            <th>Title</th>
            <th>Description</th>
            <th>Due Date</th>
            <th>Priority</th>
            <th></th>
        </tr>
        </thead>
        <tbody>
        {{#each issues}}
            <tr>
                <td>{{title}}</td>
                <td>{{description}}</td>
                <td>{{dueDateFormatted}}</td>
                <td>
                    {{#if priorityHigh}}
                        <span class="label label-danger">{{priority}}</span>
                    {{/if}}
                    {{#if priorityMedium}}
                        <span class="label label-warning">{{priority}}</span>
                    {{/if}}
                    {{#if priorityLow}}
                        <span class="label label-success">{{priority}}</span>
                    {{/if}}
                </td>
                <td>
                    {{#linkTo route='editIssue'}}
                        <i class="fa fa-pencil-square-o"></i>
                    {{/linkTo}}
                </td>
            </tr>
        {{/each}}
        </tbody>
    </table>
</template>
```

##路由配置
```routes.js
Router.configure({
  layoutTemplate: 'MasterLayout',
  loadingTemplate: 'Loading',
  notFoundTemplate: 'NotFound'
});

Router.route('/', {
  name: 'home',
  controller: 'HomeController',
  action: 'action',
  where: 'client'
});


Router.route('insert_issue', {
  name: 'insertIssue',
  controller: 'IssuesController',
  action: 'insert',
  where: 'client'
});

Router.route('issues_list', {
  name: 'issuesList',
  controller: 'IssuesController',
  action: 'list',
  where: 'client'
});

Router.route('/issues/:_id', {
  name: 'editIssue',
  controller: 'IssuesController',
  action: 'edit',
  where: 'client'
});

Router.onBeforeAction(function() {
    if (!Meteor.user()) {
        this.render('AccessDenied');
    } else
    {
        this.next();
    }
}, {only: ['issuesList', 'insertIssue']});

```

```issues_controller.js
IssuesController = RouteController.extend({
    subscriptions: function () {
        this.subscribe('issues', Meteor.userId());
    },
    data: function () {
        return Issues.findOne({_id: this.params._id});
    },
    insert: function () {
        this.render('InsertIssue', {});
    },
    list: function() {
        this.render('IssuesList', {});
    },
    edit: function() {
        this.render('EditIssue', {});
    }
});
```






