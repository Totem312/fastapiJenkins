pipeline {
    agent any

    environment {
        // Переменные окружения для удобства
        PYTHON_VERSION = '3.9'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Клонирован репозиторий FastAPI, версия: ${env.BRANCH_NAME}"
                sh 'ls -la' // Показываем содержимое папки для отладки
            }
        }

        stage('Setup Python') {
            steps {
                echo "Настройка Python ${PYTHON_VERSION}"
                // Проверяем версию Python
                sh 'python3 --version'
                
                // Создаем виртуальное окружение
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install wheel
                '''
            }
        }

        stage('Install Dependencies (Old Version)') {
            steps {
                echo "Установка зависимостей для FastAPI 0.100.0"
                sh '''
                    . venv/bin/activate
                    # Устанавливаем сам FastAPI в режиме разработки
                    # Для версии 0.100.0 используем [all] вместо [standard]
                    pip install -e .[all]
                '''
            }
        }

        stage('Run Tests') {
            steps {
                echo "Запуск тестов для версии 0.100.0"
                sh '''
                    . venv/bin/activate
                    # Устанавливаем дополнительные зависимости для тестов
                    pip install pytest pytest-cov httpx
                    
                    # Запускаем тесты с подробным выводом
                    pytest tests/ -v --tb=short
                '''
            }
        }

        stage('Version Check') {
            steps {
                echo "Проверяем, что действительно установлена версия 0.100.0"
                sh '''
                    . venv/bin/activate
                    python -c "import fastapi; print(f'FastAPI version: {fastapi.__version__}')"
                '''
            }
        }
    }

    post {
        always {
            echo "Сборка завершена. Очистка виртуального окружения..."
            sh 'rm -rf venv || true'
            archiveArtifacts artifacts: '**/test-reports/*.xml', allowEmptyArchive: true
        }
        success {
            echo "✅ Все тесты для FastAPI 0.100.0 прошли успешно!"
        }
        failure {
            echo "❌ Тесты не прошли. Проверьте логи выше."
            // Можно отправить уведомление в Slack/Telegram
        }
    }
}
