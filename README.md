@file:OptIn(ExperimentalMaterial3Api::class)

package com.example.kotest

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.itemsIndexed
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.TextFieldValue
import androidx.compose.ui.unit.dp
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import com.example.kotest.ui.theme.KoTestTheme


class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            KoTestTheme {
                val navController = rememberNavController()
                val posts = remember { mutableStateListOf<Post>() }

                NavHost(navController = navController, startDestination = "board") {
                    composable("board") { BoardScreen(navController, posts) }
                    composable("write") { WriteScreen(navController, posts) }
                    composable("detail/{index}") { backStack ->
                        val index = backStack.arguments?.getString("index")?.toIntOrNull() ?: 0
                        DetailScreen(navController, posts[index])
                    }
                }
            }
        }
    }
}
@Composable
fun BoardScreen(navController: NavHostController, posts: List<Post>) {
    Scaffold(
        floatingActionButton = {
            FloatingActionButton(
                onClick = { navController.navigate("write") },
                containerColor = MaterialTheme.colorScheme.primary
            ) {
                Text("+")
            }
        }
    ) { inner ->
        Column(
            Modifier
                .padding(inner)
                .padding(16.dp)
                .fillMaxSize()
        ) {
            Text("📋 게시판", style = MaterialTheme.typography.titleLarge)
            Spacer(Modifier.height(8.dp))
            LazyColumn(Modifier.weight(1f)) {
                itemsIndexed(posts) { index, post ->
                    TextButton(
                        onClick = { navController.navigate("detail/$index") },
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Text(post.title, style = MaterialTheme.typography.titleMedium)
                    }
                    Divider()
                }
            }
        }
    }
}
@Composable
fun WriteScreen(navController: NavHostController, posts: MutableList<Post>) {
    var title by remember { mutableStateOf(TextFieldValue("")) }
    var content by remember { mutableStateOf(TextFieldValue("")) }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("새 글 작성") },
                navigationIcon = {
                    IconButton(onClick = { navController.popBackStack() }) {
                        Text("←")
                    }
                }
            )
        }
    ) { inner ->
        Column(
            Modifier
                .padding(inner)
                .padding(16.dp)
                .fillMaxSize()
        ) {
            OutlinedTextField(
                value = title,
                onValueChange = { title = it },
                label = { Text("제목") },
                modifier = Modifier.fillMaxWidth()
            )
            Spacer(Modifier.height(8.dp))
            OutlinedTextField(
                value = content,
                onValueChange = { content = it },
                label = { Text("내용") },
                modifier = Modifier
                    .fillMaxWidth()
                    .height(150.dp)
            )
            Spacer(Modifier.height(12.dp))
            Button(onClick = {
                if (title.text.isNotBlank() && content.text.isNotBlank()) {
                    posts.add(Post(title.text, content.text))
                    navController.popBackStack() // 게시판으로 돌아감
                }
            }) {
                Text("저장")
            }
        }
    }
}
@Composable
fun DetailScreen(navController: NavHostController, post: Post) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(post.title) },
                navigationIcon = {
                    IconButton(onClick = { navController.popBackStack() }) {
                        Text("←")
                    }
                }
            )
        }
    ) { inner ->
        Column(
            Modifier
                .padding(inner)
                .padding(16.dp)
                .fillMaxSize()
        ) {
            Text(post.title, style = MaterialTheme.typography.titleLarge)
            Spacer(Modifier.height(12.dp))
            Text(post.content, style = MaterialTheme.typography.bodyLarge)
        }
    }
}
