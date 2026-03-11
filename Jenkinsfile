node {
    // Menarik kode dari GitHub (Fodly18/test-jenkins)
    checkout scm

    // 1. Build Stage
    stage("Build") {
        // Menggunakan composer terbaru yang mendukung PHP 8.4
        docker.image('composer:latest').inside('-u root') { 
            // Mengatasi error 'dubious ownership'
            sh 'git config --global --add safe.directory /var/lib/jenkins/workspace/laravel-dev'
            sh 'composer install --no-interaction --prefer-dist'
        }
    }

    stage("Testing") {
        // Menggunakan PHP 8.4 sesuai permintaan error tadi
        docker.image('php:8.4-cli').inside('-u root') {
            sh 'cp .env.example .env'
            sh 'php artisan key:generate'
            sh 'php artisan test'
        }
    }

    // 3. Deploy Stage
    stage("Deploy") {
        // Pastikan plugin SSH Agent sudah kamu instal di Jenkins tadi
        docker.image('instrumentisto/rsync-ssh').inside('-u root') {
            // 'ssh-prod' adalah ID Kredensial yang harus kamu buat di Jenkins
            sshagent (credentials: ['ssh-prod']) {
                sh 'mkdir -p ~/.ssh'
                // Pastikan variabel PROD_HOST sudah didaftarkan di Jenkins Environment
                sh "ssh-keyscan -H ${env.PROD_HOST} >> ~/.ssh/known_hosts"
                // Perhatikan path ./ (root) jika file Laravel tidak di dalam subfolder
                sh "rsync -rav --delete ./ ubuntu@${env.PROD_HOST}:/home/ubuntu/laravel-app/ --exclude=.env --exclude=storage --exclude=.git --exclude=vendor"
            }
        }
    }
}
